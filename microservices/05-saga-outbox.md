# Saga и Outbox

> Классический ACID не работает через несколько сервисов с разными БД. Saga решает согласованность бизнес-процесса через компенсации, Outbox и Inbox решают согласованность между локальной БД и message broker.

## Содержание
- [Проблема распределённых транзакций](#проблема-распределённых-транзакций)
- [Saga — Choreography](#saga--choreography)
- [Saga — Orchestration](#saga--orchestration)
- [Outbox Pattern](#outbox-pattern)
- [Inbox Pattern (Idempotent Consumer)](#inbox-pattern-idempotent-consumer)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## Проблема распределённых транзакций

Классический ACID невозможен через несколько сервисов с разными БД. 2PC (Two-Phase Commit) существует, но: coordinator — single point of failure; блокировки на время всего протокола; не работает через Kafka/Message Broker.

**Решение — Saga Pattern:** последовательность локальных транзакций с компенсациями вместо одной распределённой транзакции.

## Saga — Choreography

Сервисы реагируют на события и публикуют следующее — без центрального координатора (см. [Choreography vs Orchestration](./02-communication.md#choreography-vs-orchestration)).

```csharp
// Order Service: публикует событие при создании заказа
public class CreateOrderHandler
{
    public async Task Handle(CreateOrderCommand cmd)
    {
        var order = Order.Create(cmd.UserId, cmd.Items);
        await _orderRepository.AddAsync(order);
        await _outbox.AddAsync(new OrderCreatedEvent(order.Id, order.Total)); // атомарно с сохранением заказа
        await _unitOfWork.CommitAsync();
    }
}

// Payment Service: реагирует на OrderCreated
public class OrderCreatedHandler : IEventHandler<OrderCreatedEvent>
{
    public async Task Handle(OrderCreatedEvent evt)
    {
        var payment = await _paymentGateway.ChargeAsync(evt.UserId, evt.Amount);

        if (payment.Succeeded)
            await _outbox.AddAsync(new PaymentProcessedEvent(evt.OrderId));
        else
            await _outbox.AddAsync(new PaymentFailedEvent(evt.OrderId, payment.Error));

        await _unitOfWork.CommitAsync();
    }
}

// Order Service: компенсация при неуспехе
public class PaymentFailedHandler : IEventHandler<PaymentFailedEvent>
{
    public async Task Handle(PaymentFailedEvent evt)
    {
        var order = await _orderRepository.FindAsync(evt.OrderId);
        order.Cancel("Payment failed");
        await _unitOfWork.CommitAsync();
        // Публикуем OrderCancelled → Inventory Service отменяет резерв
    }
}
```

## Saga — Orchestration

Центральный координатор явно управляет шагами и компенсациями.

```csharp
public class OrderPlacementSaga
{
    private readonly IPaymentService _payment;
    private readonly IInventoryService _inventory;
    private readonly INotificationService _notification;

    public async Task<SagaResult> ExecuteAsync(PlaceOrderCommand cmd, CancellationToken ct)
    {
        var payment = await _payment.ChargeAsync(cmd.UserId, cmd.Amount, ct); // Step 1
        if (!payment.Succeeded)
            return SagaResult.Failed($"Payment failed: {payment.Error}");

        var inventory = await _inventory.ReserveAsync(cmd.Items, ct); // Step 2
        if (!inventory.Succeeded)
        {
            await _payment.RefundAsync(payment.TransactionId, cmd.Amount, ct); // Compensate Step 1
            return SagaResult.Failed("Inventory unavailable");
        }

        await _notification.SendOrderConfirmationAsync(cmd.UserId, cmd.OrderId, ct); // Step 3, best-effort, без компенсации

        return SagaResult.Completed();
    }
}
```

## Outbox Pattern

Решает проблему dual write между БД и Message Broker: если сначала закоммитить в БД, а потом отправить в брокер отдельным вызовом, между этими двумя действиями возможен сбой — событие потеряется или продублируется.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<OutboxMessage> OutboxMessages => Set<OutboxMessage>();
}

public class OutboxMessage
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Type { get; set; } = default!;
    public string Payload { get; set; } = default!;
    public DateTimeOffset CreatedAt { get; set; } = DateTimeOffset.UtcNow;
    public DateTimeOffset? ProcessedAt { get; set; }
    public int RetryCount { get; set; }
}

// Запись события вместе с бизнес-данными в одной транзакции
public async Task CreateOrderAsync(CreateOrderCommand cmd, CancellationToken ct)
{
    var order = Order.Create(cmd.UserId, cmd.Items);
    var evt = new OrderCreatedEvent(order.Id, order.Total);

    _db.Orders.Add(order);
    _db.OutboxMessages.Add(new OutboxMessage
    {
        Type = nameof(OrderCreatedEvent),
        Payload = JsonSerializer.Serialize(evt)
    });

    await _db.SaveChangesAsync(ct); // атомарно!
}

// Relay: фоновый worker публикует outbox в Kafka
public class OutboxRelayWorker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await ProcessBatchAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromMilliseconds(500), stoppingToken);
        }
    }

    private async Task ProcessBatchAsync(CancellationToken ct)
    {
        using var scope = _scopeFactory.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        var producer = scope.ServiceProvider.GetRequiredService<IEventProducer>();

        // Оптимистичная блокировка через FOR UPDATE SKIP LOCKED
        var messages = await db.OutboxMessages
            .Where(m => m.ProcessedAt == null && m.RetryCount < 5)
            .OrderBy(m => m.CreatedAt)
            .Take(50)
            .ToListAsync(ct);

        foreach (var msg in messages)
        {
            try
            {
                await producer.PublishAsync(msg.Type, msg.Payload, ct);
                msg.ProcessedAt = DateTimeOffset.UtcNow;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to publish outbox message {Id}", msg.Id);
                msg.RetryCount++;
            }
        }

        await db.SaveChangesAsync(ct);
    }
}
```

## Inbox Pattern (Idempotent Consumer)

Inbox — зеркальный к Outbox: гарантирует, что входящее сообщение обработано **ровно один раз**, даже если брокер доставил его повторно (at-least-once delivery).

```csharp
public class InboxProcessor
{
    public async Task ProcessAsync(string messageId, string type, string payload, CancellationToken ct)
    {
        var inserted = await _db.Database.ExecuteSqlRawAsync(
            """
            INSERT INTO inbox_messages (id, processed_at)
            VALUES ({0}, NOW())
            ON CONFLICT (id) DO NOTHING
            """, messageId, ct); // атомарно: проверить + вставить запись об обработке

        if (inserted == 0)
        {
            _logger.LogInformation("Duplicate message {MessageId}, skipping", messageId);
            return;
        }

        await DispatchAsync(type, payload, ct); // обрабатываем только если INSERT успешен
    }
}
```

## Подводные камни

- Saga Choreography без Outbox — публикация события отдельным вызовом после коммита БД теряет событие при падении между этими двумя шагами; вся цепочка компенсаций так и не запустится.
- Saga Orchestration без явного отслеживания состояния каждого шага — при падении оркестратора посреди процесса неясно, какие компенсации уже выполнены, а какие нет; нужен персистентный state (Saga State в БД, не только в памяти).
- Компенсирующая транзакция — это не `ROLLBACK`. Если платёж уже прошёл и покупатель получил письмо, компенсация — это `Refund`, а не отмена «как будто не было» (эффекты вовне уже произошли).
- Inbox без TTL/очистки таблицы — растёт бесконечно; на практике нужна политика ретенции (например, хранить ключи дедупликации 7-30 дней).

## См. также

- [Коммуникация между сервисами](./02-communication.md) — Choreography vs Orchestration
- [Паттерны данных](./03-data-patterns.md) — согласованность между Database per Service
- [Reliability и Polly](./04-reliability-polly.md) — retry на уровне отдельного вызова vs saga-компенсация на уровне процесса

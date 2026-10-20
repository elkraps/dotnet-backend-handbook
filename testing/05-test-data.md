# Данные для тестов

> Builder, Object Mother и генераторы без случайных падений.

## Содержание

- [Test Data Builder](#test-data-builder)
- [Object Mother](#object-mother)
- [Генераторы данных](#генераторы-данных)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## Test Data Builder

Builder решает проблему «хрупких тестов»: когда в конструктор добавляется новое поле — ломаются все тесты, где объект создаётся напрямую.

```csharp
/// <summary>
/// Builds Order test fixtures with sensible defaults.
/// Override only what matters for a specific test.
/// </summary>
public class OrderBuilder
{
    private Guid _id = Guid.NewGuid();
    private decimal _total = 100m;
    private string _currency = "RUB";
    private Guid _customerId = Guid.NewGuid();
    private OrderStatus _status = OrderStatus.Pending;

    public OrderBuilder WithId(Guid id) { _id = id; return this; }
    public OrderBuilder WithTotal(decimal total) { _total = total; return this; }
    public OrderBuilder WithCurrency(string currency) { _currency = currency; return this; }
    public OrderBuilder WithStatus(OrderStatus status) { _status = status; return this; }
    public OrderBuilder WithCustomer(Guid customerId) { _customerId = customerId; return this; }

    public Order Build() => new Order
    {
        Id = _id,
        Total = _total,
        Currency = _currency,
        CustomerId = _customerId,
        Status = _status
    };
}

// Использование — тест говорит только о том, что важно
[Fact]
public void Refund_CompletedOrder_ChangesStatusToRefunded()
{
    var order = new OrderBuilder()
        .WithStatus(OrderStatus.Completed)
        .Build();

    order.Refund();

    Assert.Equal(OrderStatus.Refunded, order.Status);
}
```

## Object Mother

Object Mother — фабрика готовых «персонажей» для тестов. Проще Builder, подходит когда вариантов немного.

```csharp
/// <summary>
/// Provides canonical Order instances for tests.
/// </summary>
public static class OrderMother
{
    public static Order New() => new Order
    {
        Id = Guid.Parse("11111111-0000-0000-0000-000000000001"),
        Total = 100m,
        Currency = "RUB",
        Status = OrderStatus.Pending
    };

    public static Order Completed() => new Order
    {
        Id = Guid.Parse("22222222-0000-0000-0000-000000000002"),
        Total = 250m,
        Currency = "USD",
        Status = OrderStatus.Completed
    };

    public static Order OverThreshold() => new Order
    {
        Id = Guid.Parse("33333333-0000-0000-0000-000000000003"),
        Total = 10_001m,  // превышает лимит 10 000
        Currency = "RUB",
        Status = OrderStatus.Pending
    };
}

// Использование
[Fact]
public void Approve_OrderOverThreshold_RequiresManualReview()
{
    var order = OrderMother.OverThreshold();
    var result = _service.Approve(order);
    Assert.Equal(ApprovalStatus.PendingReview, result.Status);
}
```

**Когда Builder, когда Mother:**
- Mother — когда нужны 3–5 фиксированных пресетов
- Builder — когда нужна гибкая настройка разных комбинаций

## Генераторы данных

**AutoFixture** — генерирует объекты с заполненными свойствами:

```csharp
var fixture = new Fixture();
var order = fixture.Create<Order>(); // все поля заполнены случайными данными
var orders = fixture.CreateMany<Order>(10).ToList();
```

**Bogus** — генерация реалистичных данных:

```csharp
var faker = new Faker<Customer>()
    .RuleFor(c => c.Name, f => f.Name.FullName())
    .RuleFor(c => c.Email, f => f.Internet.Email())
    .RuleFor(c => c.Phone, f => f.Phone.PhoneNumber());

var customer = faker.Generate();
```

## Подводные камни

Фиксируй seed генератора, если он влияет на результат. Новый объект на каждый тест предотвращает разделение состояния; не переиспользуй mutable builder.

## См. также

- [Unit-тесты в .NET](./04-unit-tests.md)
- [Интеграционные тесты](./06-integration-tests.md)
- [Оглавление раздела](./README.md)

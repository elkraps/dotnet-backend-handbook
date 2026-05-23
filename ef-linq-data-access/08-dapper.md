# Dapper: micro-ORM и комбинирование с EF Core

> Dapper — тонкая обёртка над ADO.NET от команды Stack Overflow. Нет Change Tracker, нет LINQ-трансляции. Ты пишешь SQL сам, Dapper маппит результат в объекты. Лучший инструмент для сложных запросов и аналитики.

## Содержание
- [Что такое Dapper](#что-такое-dapper)
- [Query и QueryFirst](#query-и-queryfirst)
- [Execute: INSERT, UPDATE, DELETE](#execute-insert-update-delete)
- [QueryMultiple: несколько результирующих наборов](#querymultiple-несколько-результирующих-наборов)
- [Multi-mapping: JOIN → несколько объектов](#multi-mapping-join--несколько-объектов)
- [Хранимые процедуры](#хранимые-процедуры)
- [Когда Dapper лучше EF Core](#когда-dapper-лучше-ef-core)
- [Когда EF Core лучше Dapper](#когда-ef-core-лучше-dapper)
- [Сравнительная таблица](#сравнительная-таблица)
- [Паттерн: EF Core + Dapper в одном репозитории](#паттерн-ef-core--dapper-в-одном-репозитории)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

---

## Что такое Dapper

Dapper расширяет `IDbConnection` extension-методами. Нет DbContext, нет модели, нет миграций.

```csharp
// Подключение
using Dapper;
using Npgsql;

using var connection = new NpgsqlConnection(connectionString);
// Соединение открывается автоматически при первом запросе
// или можно открыть явно: await connection.OpenAsync()
```

**Маппинг по соглашению:** имя колонки SQL → имя свойства C# (case-insensitive). Если не совпадает — используй `AS` в SQL или кастомный `ColumnAttribute`.

---

## Query и QueryFirst

```csharp
/// <summary>
/// Loads active orders matching criteria using Dapper.
/// </summary>
public async Task<IEnumerable<OrderDto>> LoadActiveOrders(
    IDbConnection connection, decimal minTotal, int limit)
{
    const string sql = """
        SELECT o."Id", o."Total", o."Status", c."Name" AS "CustomerName"
        FROM "Orders" o
        INNER JOIN "Customers" c ON c."Id" = o."CustomerId"
        WHERE o."Status" = @Status AND o."Total" > @MinTotal
        ORDER BY o."CreatedAt" DESC
        LIMIT @Limit
        """;

    // QueryAsync<T> — возвращает IEnumerable<T>
    return await connection.QueryAsync<OrderDto>(sql, new
    {
        Status = (int)OrderStatus.Active,
        MinTotal = minTotal,
        Limit = limit
    });
}

// QueryFirstOrDefaultAsync — первый или default(T)
var order = await connection.QueryFirstOrDefaultAsync<OrderDto>(
    """SELECT "Id", "Total" FROM "Orders" WHERE "Id" = @Id""",
    new { Id = 42 });

// QuerySingleAsync — ровно одна строка, иначе исключение
var order = await connection.QuerySingleAsync<OrderDto>(
    """SELECT "Id", "Total" FROM "Orders" WHERE "Id" = @Id""",
    new { Id = 42 });
// InvalidOperationException если 0 или >1 строк
```

**Динамические параметры:**

```csharp
// DynamicParameters — когда набор параметров неизвестен заранее
var parameters = new DynamicParameters();
parameters.Add("@Status", (int)OrderStatus.Active);

if (minTotal.HasValue)
    parameters.Add("@MinTotal", minTotal.Value);

var orders = await connection.QueryAsync<OrderDto>(sql, parameters);
```

---

## Execute: INSERT, UPDATE, DELETE

```csharp
// INSERT — возвращает количество затронутых строк
int affected = await connection.ExecuteAsync(
    """
    INSERT INTO "Orders" ("CustomerId", "Total", "Status", "CreatedAt")
    VALUES (@CustomerId, @Total, @Status, @CreatedAt)
    """,
    new
    {
        CustomerId = 1,
        Total = 99.99m,
        Status = (int)OrderStatus.Draft,
        CreatedAt = DateTime.UtcNow
    });

// Batch INSERT — Dapper автоматически итерирует по коллекции
var items = new[]
{
    new { OrderId = 1, ProductId = 10, Quantity = 2, Price = 29.99m },
    new { OrderId = 1, ProductId = 20, Quantity = 1, Price = 49.99m },
    new { OrderId = 1, ProductId = 30, Quantity = 5, Price = 9.99m  }
};

// Выполняет INSERT для каждого элемента коллекции (N отдельных INSERT)
int totalAffected = await connection.ExecuteAsync(
    """INSERT INTO "OrderItems" ("OrderId", "ProductId", "Quantity", "Price") VALUES (@OrderId, @ProductId, @Quantity, @Price)""",
    items);

// UPDATE
await connection.ExecuteAsync(
    """UPDATE "Orders" SET "Status" = @Status WHERE "Id" = @Id""",
    new { Status = (int)OrderStatus.Confirmed, Id = 42 });

// DELETE
await connection.ExecuteAsync(
    """DELETE FROM "Orders" WHERE "Id" = @Id""",
    new { Id = 42 });
```

**Получить сгенерированный Id после INSERT (PostgreSQL):**

```csharp
var newId = await connection.ExecuteScalarAsync<int>(
    """
    INSERT INTO "Orders" ("CustomerId", "Total", "Status")
    VALUES (@CustomerId, @Total, @Status)
    RETURNING "Id"
    """,
    new { CustomerId = 1, Total = 99.99m, Status = 0 });
```

---

## QueryMultiple: несколько результирующих наборов

Один round-trip к БД, несколько наборов данных.

```csharp
/// <summary>
/// Loads complete order details with items and payments in a single database round-trip.
/// </summary>
public async Task<OrderDetailDto?> LoadDetail(IDbConnection connection, int orderId)
{
    const string sql = """
        SELECT * FROM "Orders" WHERE "Id" = @Id;
        SELECT * FROM "OrderItems" WHERE "OrderId" = @Id;
        SELECT * FROM "Payments" WHERE "OrderId" = @Id;
        """;

    using var multi = await connection.QueryMultipleAsync(sql, new { Id = orderId });

    var order = await multi.ReadSingleOrDefaultAsync<OrderDto>();
    if (order is null) return null;

    var items = (await multi.ReadAsync<OrderItemDto>()).ToList();
    var payments = (await multi.ReadAsync<PaymentDto>()).ToList();

    return new OrderDetailDto
    {
        Order = order,
        Items = items,
        Payments = payments
    };
}
// Один SQL-запрос → три набора данных → три объекта в C#
```

---

## Multi-mapping: JOIN → несколько объектов

Маппинг одной строки JOIN-результата в несколько C# объектов.

```csharp
// splitOn — имя колонки, с которой начинается второй объект
var orders = await connection.QueryAsync<OrderDto, CustomerDto, OrderDto>(
    """
    SELECT o."Id", o."Total", o."Status",
           c."Id", c."Name", c."Email"
    FROM "Orders" o
    INNER JOIN "Customers" c ON c."Id" = o."CustomerId"
    ORDER BY o."Id"
    """,
    (order, customer) =>
    {
        order.Customer = customer;
        return order;
    },
    splitOn: "Id");  // второй Id — это начало Customer данных

// Три объекта в строке: splitOn: "CustomerId,ProductId"
var results = await connection.QueryAsync<A, B, C, A>(
    sql,
    (a, b, c) => { a.B = b; a.C = c; return a; },
    splitOn: "BId,CId");
```

---

## Хранимые процедуры

```csharp
// Вызов хранимой процедуры
var report = await connection.QueryAsync<ReportDto>(
    "sp_GenerateMonthlyReport",
    new { Year = 2025, Month = 12 },
    commandType: CommandType.StoredProcedure);

// С Output-параметром
var parameters = new DynamicParameters();
parameters.Add("@OrderId", 42);
parameters.Add("@TotalAmount", dbType: DbType.Decimal, direction: ParameterDirection.Output);

await connection.ExecuteAsync("sp_CalculateOrderTotal", parameters,
    commandType: CommandType.StoredProcedure);

decimal total = parameters.Get<decimal>("@TotalAmount");
```

---

## Когда Dapper лучше EF Core

**1. Сложные SQL с оконными функциями и CTE:**

```csharp
// Window functions — проще написать в raw SQL
const string sql = """
    WITH ranked AS (
        SELECT o.*,
               ROW_NUMBER() OVER (PARTITION BY o."CustomerId" ORDER BY o."Total" DESC) AS rn
        FROM "Orders" o
        WHERE o."Status" = @Status
    )
    SELECT "Id", "CustomerId", "Total", "CreatedAt"
    FROM ranked
    WHERE rn <= 3
    """;

var topOrders = await connection.QueryAsync<OrderDto>(sql, new { Status = 1 });
// EF Core может не суметь транслировать ROW_NUMBER() с PARTITION BY в LINQ
```

**2. Максимальная производительность** — Dapper на 10–30% быстрее EF Core для SELECT из-за отсутствия Change Tracker, snapshot и Expression Tree трансляции.

**3. Аналитические запросы с GROUP BY, HAVING, подзапросами:**

```csharp
const string sql = """
    SELECT c."Country",
           COUNT(DISTINCT c."Id") AS "CustomerCount",
           COUNT(o."Id") AS "OrderCount",
           AVG(o."Total") AS "AvgOrderValue",
           SUM(o."Total") AS "TotalRevenue"
    FROM "Customers" c
    LEFT JOIN "Orders" o ON o."CustomerId" = c."Id" AND o."Status" = 1
    WHERE c."IsActive" = true
    GROUP BY c."Country"
    HAVING COUNT(o."Id") > 5
    ORDER BY "TotalRevenue" DESC
    """;
```

**4. Legacy БД** со сложными схемами, которые сложно отразить в модели EF Core.

**5. Bulk INSERT через COPY (PostgreSQL):**

```csharp
// Dapper + NpgsqlBinaryImporter для очень быстрой вставки
await using var writer = await connection.BeginBinaryImportAsync(
    "COPY orders (customer_id, total, status) FROM STDIN (FORMAT BINARY)");

foreach (var order in largeDataSet)
{
    await writer.StartRowAsync();
    await writer.WriteAsync(order.CustomerId);
    await writer.WriteAsync(order.Total);
    await writer.WriteAsync((int)order.Status);
}

await writer.CompleteAsync();
```

---

## Когда EF Core лучше Dapper

1. **CRUD-операции** — EF генерирует правильный SQL, меньше ошибок в runtime
2. **Миграции** — схема базы данных синхронизируется с кодом
3. **Change Tracking** — UPDATE только изменённых полей автоматически
4. **LINQ-композиция** — динамические запросы без конкатенации строк
5. **Типобезопасность** — ошибки в запросе видны при компиляции
6. **Навигационные свойства** — удобная работа со связанными данными
7. **Тестируемость** — SQLite InMemory провайдер для unit-тестов

---

## Сравнительная таблица

| Критерий | EF Core | Dapper |
|----------|---------|--------|
| Тип | Full ORM | Micro ORM |
| SQL-генерация | Автоматическая из LINQ | Ручная |
| Change Tracking | Да | Нет |
| Миграции | Да | Нет |
| Производительность (чтение) | Хорошая | Отличная (+10–30%) |
| Сложные запросы | Ограниченно | Любой SQL |
| Типобезопасность | Compile-time (LINQ) | Runtime (строки SQL) |
| Unit of Work | Встроенный | Нужна ручная реализация |
| Lazy Loading | Да | Нет |
| Bulk-операции | EF Core 7+ (ExecuteUpdate) | Нативная поддержка |
| Кривая обучения | Высокая | Низкая |
| Размер пакета | ~5 MB | ~300 KB |
| Тестируемость | InMemory/SQLite провайдеры | Нужна реальная БД |

---

## Паттерн: EF Core + Dapper в одном репозитории

```csharp
/// <summary>
/// Order repository combining EF Core for writes and Dapper for complex reads.
/// EF Core: domain operations with change tracking and business rule enforcement.
/// Dapper: analytics and performance-critical read queries.
/// </summary>
public class OrderRepository
{
    private readonly AppDbContext _context;
    private readonly IDbConnection _connection;

    public OrderRepository(AppDbContext context, IDbConnection connection)
    {
        _context = context;
        _connection = connection;
    }

    /// <summary>
    /// Creates an order using EF Core — change tracking, validation, navigation properties.
    /// </summary>
    public async Task Create(Order order, CancellationToken token)
    {
        _context.Orders.Add(order);
        await _context.SaveChangesAsync(token);
    }

    /// <summary>
    /// Updates order status using EF Core — only changed fields go to UPDATE.
    /// </summary>
    public async Task UpdateStatus(int orderId, OrderStatus status, CancellationToken token)
    {
        var order = await _context.Orders.FindAsync(new object[] { orderId }, token)
            ?? throw new InvalidOperationException($"Order {orderId} not found.");

        order.Status = status;
        await _context.SaveChangesAsync(token);
    }

    /// <summary>
    /// Loads complex monthly report using Dapper — raw SQL with window functions.
    /// </summary>
    public async Task<IEnumerable<MonthlyReportDto>> LoadMonthlyReport(int year, int month)
    {
        const string sql = """
            WITH daily AS (
                SELECT DATE_TRUNC('day', o."CreatedAt") AS "Day",
                       COUNT(*) AS "OrderCount",
                       SUM(o."Total") AS "Revenue",
                       AVG(o."Total") AS "AvgOrderValue"
                FROM "Orders" o
                WHERE EXTRACT(YEAR FROM o."CreatedAt") = @Year
                  AND EXTRACT(MONTH FROM o."CreatedAt") = @Month
                  AND o."Status" != @CancelledStatus
                GROUP BY DATE_TRUNC('day', o."CreatedAt")
            )
            SELECT * FROM daily ORDER BY "Day"
            """;

        return await _connection.QueryAsync<MonthlyReportDto>(sql, new
        {
            Year = year,
            Month = month,
            CancelledStatus = (int)OrderStatus.Cancelled
        });
    }

    /// <summary>
    /// Loads top customers by revenue using Dapper — complex aggregation query.
    /// </summary>
    public async Task<IEnumerable<CustomerRevenueDto>> LoadTopCustomers(int topN)
    {
        const string sql = """
            SELECT c."Id", c."Name", c."Country",
                   COUNT(o."Id") AS "OrderCount",
                   SUM(o."Total") AS "TotalRevenue",
                   ROW_NUMBER() OVER (ORDER BY SUM(o."Total") DESC) AS "Rank"
            FROM "Customers" c
            INNER JOIN "Orders" o ON o."CustomerId" = c."Id"
            WHERE o."Status" = @Status
            GROUP BY c."Id", c."Name", c."Country"
            ORDER BY "TotalRevenue" DESC
            LIMIT @TopN
            """;

        return await _connection.QueryAsync<CustomerRevenueDto>(sql, new
        {
            Status = (int)OrderStatus.Confirmed,
            TopN = topN
        });
    }
}

// DI registration
builder.Services.AddScoped<IDbConnection>(_ =>
    new NpgsqlConnection(connectionString));
builder.Services.AddScoped<OrderRepository>();
```

---

## Подводные камни

**SQL injection через конкатенацию.** Никогда не строй SQL через конкатенацию строк с пользовательским вводом:

```csharp
// WRONG: SQL injection
string sql = $"""SELECT * FROM "Orders" WHERE "Status" = '{userInput}'""";

// CORRECT: параметризованные запросы
string sql = """SELECT * FROM "Orders" WHERE "Status" = @Status""";
await connection.QueryAsync<OrderDto>(sql, new { Status = userInput });
```

**Batch INSERT через Execute — не bulk.** `ExecuteAsync(sql, collection)` — это N отдельных запросов, не один bulk INSERT. Для действительно быстрой вставки используй `COPY` (PostgreSQL) или `SqlBulkCopy` (SQL Server).

**Multi-mapping splitOn — хрупкий.** `splitOn` указывает имя колонки по позиции. Если порядок колонок в SELECT изменится — маппинг сломается. Называй колонки явно в SQL.

**Dapper и транзакции.** Dapper не управляет транзакциями. Передавай `IDbTransaction` явно:

```csharp
await using var tx = await connection.BeginTransactionAsync();
try
{
    await connection.ExecuteAsync(sql1, param1, transaction: tx);
    await connection.ExecuteAsync(sql2, param2, transaction: tx);
    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

---

## См. также

- [02-efcore-architecture.md](./02-efcore-architecture.md) — SaveChanges, Change Tracker, Unit of Work
- [05-n-plus-one.md](./05-n-plus-one.md) — ручная batch-загрузка как альтернатива Include

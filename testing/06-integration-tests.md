# Интеграционные тесты

> Реальные БД и Redis, ASP.NET Core pipeline и HTTP-зависимости.

## Содержание

- [Инструменты](#инструменты)
- [Интеграционные тесты](#интеграционные-тесты)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## Инструменты

**Testcontainers** — реальные зависимости в Docker без ручной настройки:

```bash
dotnet add package Testcontainers.PostgreSql
dotnet add package Testcontainers.Redis
dotnet add package Testcontainers.Kafka
```

**Respawn** — сброс БД до чистого состояния между тестами (быстрее truncate):

```bash
dotnet add package Respawn
```

**WireMock.Net** — мок HTTP-сервера:

```bash
dotnet add package WireMock.Net
```

## Интеграционные тесты

### WebApplicationFactory

`WebApplicationFactory<TProgram>` поднимает in-process ASP.NET Core приложение. Полный pipeline работает, но без реального сетевого стека.

```csharp
public class ApiFactory : WebApplicationFactory<Program>, IAsyncLifetime
{
    public PostgreSqlContainer Postgres { get; } = new PostgreSqlBuilder()
        .WithImage("postgres:16-alpine")
        .Build();

    public async Task InitializeAsync()
    {
        await Postgres.StartAsync();
        using var scope = Services.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        await context.Database.MigrateAsync();
    }

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureTestServices(services =>
        {
            services.RemoveAll<DbContextOptions<AppDbContext>>();
            services.RemoveAll<IDbContextOptionsConfiguration<AppDbContext>>();
            services.RemoveAll<AppDbContext>();
            services.AddDbContext<AppDbContext>(options =>
                options.UseNpgsql(Postgres.GetConnectionString()));
        });
    }

    async Task IAsyncLifetime.DisposeAsync()
    {
        await base.DisposeAsync();
        await Postgres.DisposeAsync();
    }
}

public class OrdersApiTests : IClassFixture<ApiFactory>
{
    private readonly ApiFactory _factory;
    public OrdersApiTests(ApiFactory factory) => _factory = factory;

    [Fact]
    public async Task CreateOrder_ValidPayload_ReturnsCreated()
    {
        using var client = _factory.CreateClient();
        var payload = new { CustomerId = Guid.NewGuid(), Total = 157.43m, Currency = "RUB" };

        using var response = await client.PostAsJsonAsync("/api/orders", payload);

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }

    [Fact]
    public async Task CreateOrder_ValidPayload_ReturnsGeneratedId()
    {
        using var client = _factory.CreateClient();
        var payload = new { CustomerId = Guid.NewGuid(), Total = 283.71m, Currency = "RUB" };

        using var response = await client.PostAsJsonAsync("/api/orders", payload);
        var body = await response.Content.ReadFromJsonAsync<CreateOrderResponse>();

        Assert.NotEqual(Guid.Empty, body!.Id);
    }
}
```

Для top-level `Program` добавь в приложение `public partial class Program { }`. Нужны `Microsoft.AspNetCore.Mvc.Testing`, `Testcontainers.PostgreSql`, EF Core Npgsql provider и пространства имён для `ConfigureTestServices`, `RemoveAll` и `IDbContextOptionsConfiguration`.

Если `AppDbContext` зарегистрирован через pooling/factory или приложение обращается к другим внешним системам при старте, замени соответствующие регистрации тоже. Заказ в примере должен соответствовать доменным инвариантам приложения.

[Интеграционные тесты ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-8.0).


### Testcontainers

```csharp
/// <summary>
/// Integration test fixture with real PostgreSQL and Redis.
/// </summary>
public class RepositoryTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _postgres = new PostgreSqlBuilder()
        .WithImage("postgres:16-alpine")
        .WithDatabase("test_db")
        .WithUsername("test")
        .WithPassword("test")
        .Build();

    private readonly RedisContainer _redis = new RedisBuilder()
        .WithImage("redis:7-alpine")
        .Build();

    public async Task InitializeAsync()
    {
        await Task.WhenAll(_postgres.StartAsync(), _redis.StartAsync());
        await MigrateDatabase(_postgres.GetConnectionString());
    }

    public async Task DisposeAsync()
    {
        await Task.WhenAll(_postgres.DisposeAsync().AsTask(), _redis.DisposeAsync().AsTask());
    }

    [Fact]
    public async Task Save_Order_CanBeRetrieved()
    {
        using var redis = await ConnectionMultiplexer.ConnectAsync(_redis.GetConnectionString());
        var repo = new CachedOrderRepository(
            new OrderRepository(_postgres.GetConnectionString()),
            redis
        );
        var order = new OrderBuilder().Build();

        await repo.Save(order);
        var cached = await repo.Find(order.Id);

        Assert.Equal(order.Id, cached!.Id);
    }

    private static async Task MigrateDatabase(string connectionString)
    {
        // Применяем миграции EF Core или SQL-скрипты
        await using var context = new AppDbContext(
            new DbContextOptionsBuilder<AppDbContext>()
                .UseNpgsql(connectionString).Options);
        await context.Database.MigrateAsync();
    }
}
```

### Respawn

Respawn удаляет данные между тестами умнее, чем `TRUNCATE ALL`: учитывает FK-зависимости, не пересоздаёт схему.

```csharp
public class OrderRepositoryTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _postgres = new PostgreSqlBuilder().Build();
    private Respawner _respawner = null!;
    private NpgsqlConnection _connection = null!;

    public async Task InitializeAsync()
    {
        await _postgres.StartAsync();
        _connection = new NpgsqlConnection(_postgres.GetConnectionString());
        await _connection.OpenAsync();

        await MigrateDatabase();

        _respawner = await Respawner.CreateAsync(_connection, new RespawnerOptions
        {
            DbAdapter = DbAdapter.Postgres,
            SchemasToInclude = new[] { "public" }
        });
    }

    public async Task DisposeAsync()
    {
        await _connection.DisposeAsync();
        await _postgres.DisposeAsync();
    }

    // Вызывается перед каждым тестом
    private async Task ResetDatabase() => await _respawner.ResetAsync(_connection);

    [Fact]
    public async Task Find_AfterSave_ReturnsOrder()
    {
        await ResetDatabase();
        var repo = new OrderRepository(_postgres.GetConnectionString());
        var order = new OrderBuilder().Build();

        await repo.Save(order);
        var result = await repo.Find(order.Id);

        Assert.Equal(order.Total, result!.Total);
    }
}
```

### WireMock

```csharp
/// <summary>
/// Tests that use WireMock to stub external payment gateway HTTP calls.
/// </summary>
public class PaymentServiceTests : IDisposable
{
    private readonly WireMockServer _server = WireMockServer.Start();

    [Fact]
    public async Task Charge_SuccessResponse_ReturnsApproved()
    {
        _server.Given(Request.Create()
                   .WithPath("/charge")
                   .UsingPost())
               .RespondWith(Response.Create()
                   .WithStatusCode(200)
                   .WithBodyAsJson(new { status = "approved", transactionId = "txn_123" }));

        var service = new PaymentGatewayClient(_server.Urls[0]);

        var result = await service.Charge("tok_valid", 100m);

        Assert.Equal("approved", result.Status);
    }

    public void Dispose() => _server.Dispose();
}
```

### InMemory-провайдер EF Core — когда можно

InMemory провайдер — не реляционная БД. Он не поддерживает:
- транзакции
- FK-ограничения
- SQL-запросы через `FromSqlRaw`
- конкурентный доступ

```csharp
// Допустимо: тест чистой доменной логики через репозиторий
// без проверки SQL-специфики
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
    .Options;
using var context = new AppDbContext(options);

// Недопустимо: тест транзакций, FK, индексов
// — используй Testcontainers
```

Успешный `Find` сам по себе не доказывает cache hit: дополнительно проверь наличие ключа либо отсутствие обращения к исходному хранилищу в отдельном тесте. Для Respawn используй отдельную БД/схему на параллельного владельца: reset общей БД может удалить данные другого теста.

## Подводные камни

Fixture разделяет ресурсы, а не гарантирует изоляцию данных. Docker должен быть доступен; контейнер запускается до создания клиента приложения.

## См. также

- [Устройство xUnit](./08-xunit-internals.md)
- [Кеширование в .NET](./../caching/06-dotnet-cache.md)
- [Оглавление раздела](./README.md)

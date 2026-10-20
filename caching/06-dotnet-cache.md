# Кеширование в .NET

> IMemoryCache, IDistributedCache, ConnectionMultiplexer и условные транзакции.

## Содержание

- [IMemoryCache — in-process кеш](#imemorycache--in-process-кеш)
- [IDistributedCache — абстракция над распределённым кешем](#idistributedcache--абстракция-над-распределённым-кешем)
- [StackExchange.Redis — прямой доступ к Redis](#stackexchangeredis--прямой-доступ-к-redis)
- [MULTI/EXEC — транзакции Redis](#multiexec--транзакции-redis)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## IMemoryCache — in-process кеш

```csharp
// Регистрация
builder.Services.AddMemoryCache(options =>
{
    options.SizeLimit = 1000;  // максимум 1000 "единиц"
});

// Использование
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly AppDbContext _db;

    public ProductService(IMemoryCache cache, AppDbContext db)
    {
        _cache = cache;
        _db = db;
    }

    public async Task<Product?> GetAsync(int id, CancellationToken ct)
    {
        // Concurrent misses may invoke this factory more than once.
        return await _cache.GetOrCreateAsync($"product:{id}", async entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
            entry.SlidingExpiration = TimeSpan.FromMinutes(2);
            entry.Size = 1;  // единицы для SizeLimit
            entry.Priority = CacheItemPriority.Normal;

            // Callback при вытеснении
            entry.RegisterPostEvictionCallback((key, value, reason, state) =>
                Console.WriteLine($"Cache entry evicted, reason: {reason}"));

            return await _db.Products.FindAsync(new object[] { id }, ct);
        });
    }

    public void Invalidate(int id)
    {
        _cache.Remove($"product:{id}");
    }
}
```

**Операции `IMemoryCache` потокобезопасны, но фабрика `GetOrCreateAsync` не получает взаимного исключения** — два потока с одним ключом могут одновременно пройти в БД. Для защиты используй `SemaphoreSlim` per key (см. [защиту от stampede](./08-cache-problems.md)).

Если задаёшь `SizeLimit`, каждая запись обязана указывать размер в тех же условных единицах. Для приложения со сторонними компонентами выдели отдельный экземпляр кеша с ограничением, чтобы их записи без `Size` не ломались.

[Документация IMemoryCache](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory).

## IDistributedCache — абстракция над распределённым кешем

```csharp
// Регистрация Redis как IDistributedCache
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "myapp:";  // префикс для всех ключей
});

// Или in-memory для тестов
builder.Services.AddDistributedMemoryCache();

// Использование (сериализация вручную)
public class CacheService
{
    private readonly IDistributedCache _cache;

    public CacheService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task<T?> GetAsync<T>(string key, CancellationToken ct)
    {
        var bytes = await _cache.GetAsync(key, ct);
        return bytes is null ? default : JsonSerializer.Deserialize<T>(bytes);
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan ttl, CancellationToken ct)
    {
        var bytes = JsonSerializer.SerializeToUtf8Bytes(value);
        await _cache.SetAsync(key, bytes,
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = ttl
            }, ct);
    }

    public async Task RemoveAsync(string key, CancellationToken ct)
        => await _cache.RemoveAsync(key, ct);
}
```

`IDistributedCache` — минимальный интерфейс: Get/Set/Refresh/Remove. Нет поддержки Redis-специфичных команд (ZSet, Lua и т.д.) — для этого нужен `IDatabase` напрямую.

## StackExchange.Redis — прямой доступ к Redis

```csharp
// ConnectionMultiplexer — создавать ОДИН раз (Singleton)
builder.Services.AddSingleton<IConnectionMultiplexer>(sp =>
{
    var config = ConfigurationOptions.Parse("localhost:6379");
    config.ConnectRetry = 5;
    config.ReconnectRetryPolicy = new ExponentialRetry(5000);
    config.AbortOnConnectFail = false;
    return ConnectionMultiplexer.Connect(config);
});

// IDatabase — получать на каждый запрос (легковесный)
public class RedisProductRepository
{
    private readonly IConnectionMultiplexer _redis;

    public RedisProductRepository(IConnectionMultiplexer redis)
    {
        _redis = redis;
    }

    private IDatabase Db => _redis.GetDatabase();

    // Cache lookup; source loading belongs to the caller.
    public async Task<Product?> GetAsync(int id, CancellationToken ct)
    {
        var key = $"product:{id}";
        var cached = await Db.StringGetAsync(key);

        if (cached.HasValue)
            return JsonSerializer.Deserialize<Product>(cached!);

        // Cache miss: caller must load from the source.
        return null;
    }

    // Sorted Set: лидерборд
    public async Task UpdateScoreAsync(string userId, double score)
    {
        await Db.SortedSetAddAsync("leaderboard", userId, score);
    }

    public async Task<SortedSetEntry[]> GetTopAsync(int count)
    {
        return await Db.SortedSetRangeByRankWithScoresAsync(
            "leaderboard", 0, count - 1, Order.Descending);
    }

    // Hash: атомарное обновление полей объекта
    public async Task UpdateUserFieldAsync(int userId, string field, string value)
    {
        await Db.HashSetAsync($"user:{userId}", field, value);
    }

    // Pipeline: несколько команд без round-trip за каждую
    public async Task<Product[]> GetManyAsync(int[] ids)
    {
        var batch = Db.CreateBatch();
        var tasks = ids.Select(id =>
            batch.StringGetAsync($"product:{id}")).ToArray();
        batch.Execute();

        var results = await Task.WhenAll(tasks);
        return results
            .Where(r => r.HasValue)
            .Select(r => JsonSerializer.Deserialize<Product>(r!)!)
            .ToArray();
    }
}
```

**ConnectionMultiplexer — почему Singleton:**
- Мультиплексирует команды по переиспользуемым соединениям; фиксированного пула на 5–10 соединений нет
- Потокобезопасен
- Дорого создавать (TLS handshake, аутентификация)
- `GetDatabase()` — легковесно, без нового соединения

## MULTI/EXEC — транзакции Redis

Redis выполняет команды между `MULTI` и `EXEC` без вмешательства команд других клиентов. `WATCH` позволяет отменить выполнение при изменении отслеживаемого ключа. Ошибка отдельной команды во время выполнения не откатывает уже выполненные команды.

В StackExchange.Redis используй `AddCondition`: клиент сам управляет `WATCH` и мультиплексированным соединением.

```csharp
var db = _redis.GetDatabase();
var key = "quota:worker:42";
var committed = false;

for (var attempt = 0; attempt < 3; attempt++)
{
    var current = await db.StringGetAsync(key);
    if (!current.HasValue || (long)current <= 0)
        break;

    var tx = db.CreateTransaction();
    tx.AddCondition(Condition.StringEqual(key, current));
    var decrement = tx.StringDecrementAsync(key);

    if (await tx.ExecuteAsync())
    {
        await decrement;
        committed = true;
        break;
    }
}
```

Ключ инициализируется отдельно. Не сбрасывай его при каждом retry. При конфликте queued tasks отменяются; retry ограничен. Для атомарного «проверить остаток и уменьшить» часто проще Lua. Не делай `await` queued-команды до `ExecuteAsync`: она ещё не отправлена.

[Транзакции StackExchange.Redis](https://seredis.dev/Transactions).


## Подводные камни

AddDistributedMemoryCache хранит данные в одном процессе. Это тестовая реализация интерфейса, а не общий кеш для нескольких instances.

## См. также

- [Stampede, penetration, avalanche и hot key](./08-cache-problems.md)
- [Интеграционные тесты](./../testing/06-integration-tests.md)
- [Оглавление раздела](./README.md)

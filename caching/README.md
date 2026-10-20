# Кеширование: стратегии, Redis и .NET

От выбора стратегии и TTL до инвалидации, защиты источника и практических сценариев Redis. Примеры показывают механизмы; ограничения каждого подхода разобраны рядом с кодом.

## Порядок чтения

| Файл | Что внутри |
|------|------------|
| [Уровни кеширования](./01-cache-levels.md) | Цена кеша, hit ratio и путь Browser → L1 → L2 → DB. |
| [Стратегии кеширования](./02-cache-strategies.md) | Cache-aside, read-through, write-through, write-behind и refresh-ahead. |
| [TTL и вытеснение](./03-expiration-eviction.md) | Absolute/sliding expiration, LRU/LFU и лимит памяти. |
| [Memcached и сравнение с Redis](./04-memcached.md) | Slab allocator, consistent hashing и выбор хранилища. |
| [Основы Redis для кеширования](./05-redis-basics.md) | Структуры данных, RDB/AOF, Pub/Sub, Sentinel, Cluster и Lua. |
| [Кеширование в .NET](./06-dotnet-cache.md) | IMemoryCache, IDistributedCache, ConnectionMultiplexer и условные транзакции. |
| [Инвалидация и консистентность](./07-invalidation-consistency.md) | Почему обновление БД и удаление кеша не исключают stale data. |
| [Stampede, penetration, avalanche и hot key](./08-cache-problems.md) | Как кеш может перегрузить источник и какие меры помогают. |
| [Практические паттерны Redis](./09-practical-patterns.md) | L1/L2, distributed leases, rate limiting и session storage. |

## Связанные разделы

- [Тестирование](../testing/README.md)
- [Async/Await](../async-await/README.md)

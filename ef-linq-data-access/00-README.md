# EF Core + LINQ: доступ к данным в .NET

EF Core — основной ORM в экосистеме .NET. LINQ — язык запросов, который EF транслирует в SQL. Здесь — внутреннее устройство Change Tracker, правила трансляции LINQ→SQL, N+1, связи, конфигурация и Dapper.

## Порядок чтения

1. [01-linq-core.md](./01-linq-core.md) — IEnumerable vs IQueryable, Expression Trees, deferred execution, streaming vs buffering, multiple enumeration
2. [02-efcore-architecture.md](./02-efcore-architecture.md) — DbContext (UoW+Repository), Change Tracker (snapshot vs notification), 5 состояний сущности, SaveChanges pipeline
3. [03-efcore-queries.md](./03-efcore-queries.md) — LINQ→SQL трансляция, client vs server evaluation, Include/ThenInclude, cartesian explosion, AsSplitQuery, Explicit/Lazy Loading
4. [04-projection-notracking.md](./04-projection-notracking.md) — Select vs Include, AsNoTracking, AsNoTrackingWithIdentityResolution, таблица производительности
5. [05-n-plus-one.md](./05-n-plus-one.md) — причины N+1, обнаружение (MiniProfiler, interceptors), все стратегии исправления
6. [06-relationships.md](./06-relationships.md) — One-to-One/Many/Many-to-Many, типы коллекций, backing fields, cascade delete
7. [07-configuration.md](./07-configuration.md) — Fluent API vs Annotations, IEntityTypeConfiguration, индексы, value conversion, global query filter, owned types
8. [08-dapper.md](./08-dapper.md) — micro-ORM, Query/Execute/QueryMultiple/multi-mapping, когда Dapper vs EF Core, паттерн комбинирования

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| IEnumerable vs IQueryable | [01-linq-core.md](./01-linq-core.md) |
| Expression Trees | [01-linq-core.md](./01-linq-core.md) |
| Deferred execution, триггеры | [01-linq-core.md](./01-linq-core.md) |
| Streaming vs Buffering операторы | [01-linq-core.md](./01-linq-core.md) |
| Multiple enumeration problem | [01-linq-core.md](./01-linq-core.md) |
| DbContext (UoW + Repository) | [02-efcore-architecture.md](./02-efcore-architecture.md) |
| IDbContextFactory | [02-efcore-architecture.md](./02-efcore-architecture.md) |
| Change Tracker snapshot vs notification | [02-efcore-architecture.md](./02-efcore-architecture.md) |
| 5 состояний сущности (Added/Unchanged/Modified/Deleted/Detached) | [02-efcore-architecture.md](./02-efcore-architecture.md) |
| SaveChanges 14-шаговый pipeline | [02-efcore-architecture.md](./02-efcore-architecture.md) |
| ExecuteUpdate / ExecuteDelete (bulk) | [02-efcore-architecture.md](./02-efcore-architecture.md) |
| LINQ→SQL pipeline | [03-efcore-queries.md](./03-efcore-queries.md) |
| Client vs Server Evaluation | [03-efcore-queries.md](./03-efcore-queries.md) |
| Include / ThenInclude | [03-efcore-queries.md](./03-efcore-queries.md) |
| Cartesian Explosion | [03-efcore-queries.md](./03-efcore-queries.md) |
| AsSplitQuery | [03-efcore-queries.md](./03-efcore-queries.md) |
| Explicit Loading (.Query() фильтрация) | [03-efcore-queries.md](./03-efcore-queries.md) |
| Lazy Loading (proxy, ILazyLoader) | [03-efcore-queries.md](./03-efcore-queries.md) |
| Select vs Include (SQL сравнение) | [04-projection-notracking.md](./04-projection-notracking.md) |
| Auto-JOIN в проекции | [04-projection-notracking.md](./04-projection-notracking.md) |
| AsNoTracking | [04-projection-notracking.md](./04-projection-notracking.md) |
| AsNoTrackingWithIdentityResolution | [04-projection-notracking.md](./04-projection-notracking.md) |
| Таблица производительности tracking/notracking | [04-projection-notracking.md](./04-projection-notracking.md) |
| N+1 через Lazy Loading | [05-n-plus-one.md](./05-n-plus-one.md) |
| Обнаружение N+1 (MiniProfiler, QueryCountInterceptor) | [05-n-plus-one.md](./05-n-plus-one.md) |
| Исправление через Include | [05-n-plus-one.md](./05-n-plus-one.md) |
| Исправление через Select | [05-n-plus-one.md](./05-n-plus-one.md) |
| Исправление через batch-загрузку | [05-n-plus-one.md](./05-n-plus-one.md) |
| One-to-One связь | [06-relationships.md](./06-relationships.md) |
| One-to-Many связь | [06-relationships.md](./06-relationships.md) |
| Many-to-Many (implicit + explicit) | [06-relationships.md](./06-relationships.md) |
| ICollection vs IReadOnlyCollection | [06-relationships.md](./06-relationships.md) |
| Что EF Core инжектирует в коллекции | [06-relationships.md](./06-relationships.md) |
| Backing Fields, PropertyAccessMode | [06-relationships.md](./06-relationships.md) |
| Cascade Delete (все варианты) | [06-relationships.md](./06-relationships.md) |
| Fluent API vs Data Annotations (приоритет) | [07-configuration.md](./07-configuration.md) |
| IEntityTypeConfiguration<T> | [07-configuration.md](./07-configuration.md) |
| Индексы (составной, фильтрованный, covering) | [07-configuration.md](./07-configuration.md) |
| Value Conversion (enum→string, list→string) | [07-configuration.md](./07-configuration.md) |
| Global Query Filter (soft delete, multi-tenancy) | [07-configuration.md](./07-configuration.md) |
| Owned Types (Value Objects) | [07-configuration.md](./07-configuration.md) |
| Dapper основные методы | [08-dapper.md](./08-dapper.md) |
| QueryMultiple (несколько наборов) | [08-dapper.md](./08-dapper.md) |
| Multi-mapping (JOIN → несколько объектов) | [08-dapper.md](./08-dapper.md) |
| EF Core vs Dapper сравнение | [08-dapper.md](./08-dapper.md) |
| Паттерн EF Core + Dapper | [08-dapper.md](./08-dapper.md) |
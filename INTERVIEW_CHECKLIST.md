# .NET Backend Interview Checklist

Чеклист вопросов для собеседований на позицию .NET Backend Developer (middle/senior).

---

## 1. ООП и основы C#

### Концепции ООП
- [ ] Что такое ООП? Назови четыре принципа (инкапсуляция, наследование, полиморфизм, абстракция) — объясни каждый с примером на C#
- [ ] Чем абстрактный класс отличается от интерфейса? Когда использовать каждый?
- [ ] Что такое полиморфизм? Чем compile-time (перегрузка) отличается от runtime (переопределение)?
- [ ] Что такое ковариантность и контравариантность в generic-типах и делегатах?
- [ ] Чем `sealed` класс отличается от обычного? Когда это важно для производительности?

### Типы данных C#
- [ ] Чем value type отличается от reference type? Где хранится каждый? — [clr-fundamentals/02-value-reference-types.md](./clr-fundamentals/02-value-reference-types.md)
- [ ] Что такое boxing/unboxing? Почему это дорого? Как избежать? — [clr-fundamentals/06-boxing.md](./clr-fundamentals/06-boxing.md)
- [ ] Чем `struct` отличается от `class` и `record`? Когда использовать struct? — [clr-fundamentals/05-struct-class-record.md](./clr-fundamentals/05-struct-class-record.md)
- [ ] Как работает `string` в .NET? Что такое string interning? Почему string иммутабельный? — [clr-fundamentals/03-string.md](./clr-fundamentals/03-string.md)
- [ ] Что такое стек и куча? Что куда попадает? — [clr-fundamentals/04-stack-heap.md](./clr-fundamentals/04-stack-heap.md)

### Generics и делегаты
- [ ] Что такое generics? Зачем они нужны? Что такое generic constraints (`where T : class`, `new()`, etc.)?
- [ ] Чем `Func<T>`, `Action<T>` и `Predicate<T>` отличаются? Когда использовать каждый?
- [ ] Что такое expression tree? Как LINQ провайдеры используют `Expression<Func<T>>`?
- [ ] Что такое `delegate` под капотом? Как работает multicast delegate?
- [ ] Чем `event` отличается от `delegate`? Что такое event accessor?

### Паттерны C# (современные возможности)
- [ ] Что такое pattern matching в C#? Примеры `switch expression`, `is` patterns, `when` guard
- [ ] Что такое nullable reference types (`?`)? Как включить и как работает flow analysis?
- [ ] Что такое `record` и `record struct`? Как работает value equality? Что генерирует компилятор?
- [ ] Что такое `ref struct` и `Span<T>`? Зачем они нужны и где используются?
- [ ] Что такое `IDisposable`/`using`? Зачем нужен `IAsyncDisposable`? — [clr-fundamentals/08-finalization-dispose.md](./clr-fundamentals/08-finalization-dispose.md)

---

## 2. CLR и рантайм

### Сборщик мусора
- [ ] Как работает GC в .NET? Что такое поколения (Gen0, Gen1, Gen2, LOH)? — [clr-fundamentals/07-gc.md](./clr-fundamentals/07-gc.md)
- [ ] Что такое финализатор? Чем отличается от `Dispose`? Почему финализатор плохо использовать? — [clr-fundamentals/08-finalization-dispose.md](./clr-fundamentals/08-finalization-dispose.md)
- [ ] Что такое Large Object Heap? Почему он не компактируется по умолчанию? — [clr-fundamentals/07-gc.md](./clr-fundamentals/07-gc.md)
- [ ] Что такое `GC.Collect()`? Когда его допустимо вызвать? Почему обычно не нужно?
- [ ] Что такое `WeakReference<T>`? Когда он полезен?

### CLR и JIT
- [ ] Что такое CLR? Что такое JIT? Чем AOT отличается от JIT? — [clr-fundamentals/01-dotnet-platform.md](./clr-fundamentals/01-dotnet-platform.md)
- [ ] Что такое assembly? Что такое MSIL/IL? Как выглядит пайплайн компиляции .NET?
- [ ] Что такое reflection? Как работает `typeof()`, `GetType()`, `Activator.CreateInstance()`?
- [ ] Что такое source generators? Когда использовать вместо reflection?
- [ ] Что такое интерфейсы под капотом в CLR? Как работает dispatch? — [clr-fundamentals/09-interfaces.md](./clr-fundamentals/09-interfaces.md)

---

## 3. Async/Await и многопоточность

### Async/Await
- [ ] Как работает `async/await` под капотом? Что такое state machine? — [async-await/03-state-machine.md](./async-await/03-state-machine.md)
- [ ] Что такое `Task` vs `ValueTask`? Когда использовать `ValueTask`? — [async-await/07-valuetask.md](./async-await/07-valuetask.md)
- [ ] Что такое `SynchronizationContext`? Зачем `ConfigureAwait(false)`? — [async-await/06-synchronization-context.md](./async-await/06-synchronization-context.md)
- [ ] Что такое `CancellationToken`? Как правильно пробрасывать через цепочку вызовов? — [async-await/08-cancellation.md](./async-await/08-cancellation.md)
- [ ] Антипаттерны async: `.Result`, `.Wait()`, `async void`, fire-and-forget — [async-await/10-antipatterns.md](./async-await/10-antipatterns.md)
- [ ] Что такое ThreadPool? Как он управляет потоками? — [async-await/01-threadpool.md](./async-await/01-threadpool.md)
- [ ] Что такое awaitable паттерн? Можно ли сделать `await` над чем-то кастомным? — [async-await/04-awaitable-pattern.md](./async-await/04-awaitable-pattern.md)

### Примитивы синхронизации
- [ ] Что такое `lock`/`Monitor`? Как работает под капотом? Что такое deadlock? — [synchronization-primitives/02-lock-monitor.md](./synchronization-primitives/02-lock-monitor.md)
- [ ] Чем `SemaphoreSlim` отличается от `Semaphore`? Когда использовать каждый? — [synchronization-primitives/03-rw-sem-event.md](./synchronization-primitives/03-rw-sem-event.md)
- [ ] Что такое `ReaderWriterLockSlim`? Когда выгоден? — [synchronization-primitives/03-rw-sem-event.md](./synchronization-primitives/03-rw-sem-event.md)
- [ ] Что такое `Interlocked`? Что такое `volatile`? Как работает memory model? — [synchronization-primitives/05-interlocked-volatile.md](./synchronization-primitives/05-interlocked-volatile.md)
- [ ] Что такое `ConcurrentDictionary`, `ConcurrentQueue`, `Channel<T>`? — [synchronization-primitives/07-concurrent-collections.md](./synchronization-primitives/07-concurrent-collections.md)
- [ ] Классические проблемы: deadlock, livelock, race condition, starvation — [synchronization-primitives/08-problems.md](./synchronization-primitives/08-problems.md)

### TPL и параллелизм
- [ ] Чем конкурентность отличается от параллелизма? — [tpl-parallel/01-concurrency-vs-parallelism.md](./tpl-parallel/01-concurrency-vs-parallelism.md)
- [ ] `Task.Run` vs `Task.Factory.StartNew` — в чём разница? — [tpl-parallel/02-task-run.md](./tpl-parallel/02-task-run.md)
- [ ] Что такое `Parallel.For`/`Parallel.ForEach`? Когда ускорит, а когда замедлит? — [tpl-parallel/03-parallel.md](./tpl-parallel/03-parallel.md)
- [ ] Что такое PLINQ? Когда использовать `.AsParallel()`? — [tpl-parallel/04-plinq.md](./tpl-parallel/04-plinq.md)
- [ ] Что такое TPL Dataflow? Когда использовать вместо `Channel<T>`? — [tpl-parallel/05-dataflow.md](./tpl-parallel/05-dataflow.md)

---

## 4. ASP.NET Core Pipeline

### Архитектура и Kestrel
- [ ] Что такое Kestrel? Зачем нужен reverse proxy (Nginx/IIS) перед ним? — [aspnet-pipeline/01-kestrel.md](./aspnet-pipeline/01-kestrel.md)
- [ ] Как выглядит жизненный цикл HTTP-запроса в ASP.NET Core? — [aspnet-pipeline/02-request-lifecycle.md](./aspnet-pipeline/02-request-lifecycle.md)

### Middleware
- [ ] Что такое middleware? Как работает конвейер? Чем `Use()`, `Run()`, `Map()` отличаются? — [aspnet-pipeline/03-middleware.md](./aspnet-pipeline/03-middleware.md)
- [ ] Каков правильный порядок middleware в `Program.cs`? Почему порядок важен? — [aspnet-pipeline/03-middleware.md](./aspnet-pipeline/03-middleware.md)
- [ ] Как написать кастомный middleware? Какой lifecycle у сервисов в middleware? — [aspnet-pipeline/03-middleware.md](./aspnet-pipeline/03-middleware.md)

### Dependency Injection
- [ ] Что такое DI в ASP.NET Core? Чем `Singleton`, `Scoped`, `Transient` отличаются? — [aspnet-pipeline/07-dependency-injection.md](./aspnet-pipeline/07-dependency-injection.md)
- [ ] Что такое captive dependency? Почему Scoped нельзя инжектить в Singleton? — [aspnet-pipeline/07-dependency-injection.md](./aspnet-pipeline/07-dependency-injection.md)
- [ ] Что такое Keyed Services (.NET 8+)? Для чего они нужны?
- [ ] Как сделать Service Locator через `IServiceProvider`? Почему это антипаттерн?

### Routing и Model Binding
- [ ] Как работает routing в ASP.NET Core? Route templates, constraints, attribute routing — [aspnet-pipeline/04-routing.md](./aspnet-pipeline/04-routing.md)
- [ ] Что такое model binding? Откуда берёт данные (query, body, route, header)? — [aspnet-pipeline/05-model-binding.md](./aspnet-pipeline/05-model-binding.md)
- [ ] Что такое фильтры (filters)? Чем отличаются от middleware? Порядок выполнения? — [aspnet-pipeline/10-filters.md](./aspnet-pipeline/10-filters.md)

### Конфигурация и хостинг
- [ ] Как работает конфигурация в ASP.NET Core? `IOptions<T>`, `IOptionsSnapshot`, `IOptionsMonitor` — [aspnet-pipeline/08-configuration.md](./aspnet-pipeline/08-configuration.md)
- [ ] Что такое `IHostedService` и `BackgroundService`? — [aspnet-pipeline/09-hosted-services.md](./aspnet-pipeline/09-hosted-services.md)
- [ ] Как глобально обрабатывать ошибки? `UseExceptionHandler` vs exception filter vs middleware — [aspnet-pipeline/06-exception-handling.md](./aspnet-pipeline/06-exception-handling.md)

---

## 5. EF Core и LINQ

### LINQ
- [ ] Чем `IEnumerable<T>` отличается от `IQueryable<T>`? Что такое deferred execution? — [ef-linq-data-access/01-linq-core.md](./ef-linq-data-access/01-linq-core.md)
- [ ] Как LINQ транслируется в SQL? Что происходит при `.ToList()`, `.First()`, `.Count()`? — [ef-linq-data-access/01-linq-core.md](./ef-linq-data-access/01-linq-core.md)

### EF Core
- [ ] Как работает Change Tracker? Что такое `EntityState`? — [ef-linq-data-access/02-efcore-architecture.md](./ef-linq-data-access/02-efcore-architecture.md)
- [ ] Что такое N+1 проблема? Как обнаружить? Как исправить через `Include` или `Select`? — [ef-linq-data-access/05-n-plus-one.md](./ef-linq-data-access/05-n-plus-one.md)
- [ ] Чем `AsNoTracking()` полезен? Когда его применять? — [ef-linq-data-access/04-projection-notracking.md](./ef-linq-data-access/04-projection-notracking.md)
- [ ] Как настраивать маппинг через Fluent API? Чем лучше Data Annotations? — [ef-linq-data-access/07-configuration.md](./ef-linq-data-access/07-configuration.md)
- [ ] Виды связей в EF Core: one-to-one, one-to-many, many-to-many — [ef-linq-data-access/06-relationships.md](./ef-linq-data-access/06-relationships.md)
- [ ] Когда использовать Dapper вместо EF Core? — [ef-linq-data-access/08-dapper.md](./ef-linq-data-access/08-dapper.md)
- [ ] Как применять миграции в production? Code-first vs DB-first. Стратегии zero-downtime
- [ ] Что такое оптимистичный параллелизм? Как реализовать через `RowVersion`/`ConcurrencyToken`?
- [ ] `ExecuteUpdateAsync`/`ExecuteDeleteAsync` (.NET 7+) — зачем нужны? Отличие от обычного update

---

## 6. База данных

### Транзакции и изоляция
- [ ] Что такое транзакция? ACID свойства — [database-internals/01-transaction-isolation.md](./database-internals/01-transaction-isolation.md)
- [ ] Уровни изоляции: Read Uncommitted, Read Committed, Repeatable Read, Serializable — [database-internals/01-transaction-isolation.md](./database-internals/01-transaction-isolation.md)
- [ ] Аномалии: dirty read, non-repeatable read, phantom read — [database-internals/01-transaction-isolation.md](./database-internals/01-transaction-isolation.md)
- [ ] Что такое MVCC? Как PostgreSQL реализует неблокирующие чтения? — [database-internals/02-mvcc.md](./database-internals/02-mvcc.md)
- [ ] Виды блокировок в PostgreSQL: row-level, table-level, advisory locks — [database-internals/03-locking.md](./database-internals/03-locking.md)

### Индексы и оптимизация
- [ ] Что такое B-tree индекс? Как работает? Для каких запросов подходит? — [database-internals/04-indexes.md](./database-internals/04-indexes.md)
- [ ] Когда индекс не используется? Что такое index scan vs seq scan? — [database-internals/04-indexes.md](./database-internals/04-indexes.md)
- [ ] Что такое GIN/GiST индексы? Когда нужны? — [database-internals/04-indexes.md](./database-internals/04-indexes.md)
- [ ] Как читать `EXPLAIN ANALYZE`? Что такое seq scan, index scan, hash join? — [database-internals/09-query-optimization.md](./database-internals/09-query-optimization.md)
- [ ] Алгоритмы JOIN: Nested Loop, Hash Join, Merge Join — когда какой выбирает планировщик? — [database-internals/05-join-algorithms.md](./database-internals/05-join-algorithms.md)

### Масштабирование
- [ ] Что такое шардирование? Стратегии: hash sharding, range sharding. Минусы? — [database-internals/06-sharding.md](./database-internals/06-sharding.md)
- [ ] Что такое репликация? Master-Slave, синхронная vs асинхронная — [database-internals/07-replication.md](./database-internals/07-replication.md)
- [ ] Что такое HA? Failover, Patroni, connection pooling (PgBouncer) — [database-internals/08-high-availability.md](./database-internals/08-high-availability.md)

---

## 7. Web API Protocols

- [ ] Что такое REST? Принципы (stateless, uniform interface, HATEOAS). REST vs HTTP API — [web-api-protocols/01-rest-design.md](./web-api-protocols/01-rest-design.md)
- [ ] REST паттерны: versioning, pagination (offset vs cursor), partial updates (PATCH vs PUT) — [web-api-protocols/02-rest-patterns.md](./web-api-protocols/02-rest-patterns.md)
- [ ] Что такое gRPC? Protobuf vs JSON. Когда выбрать gRPC вместо REST? — [web-api-protocols/03-grpc-protobuf.md](./web-api-protocols/03-grpc-protobuf.md)
- [ ] Виды gRPC streaming: unary, server streaming, client streaming, bidirectional — [web-api-protocols/04-grpc-dotnet.md](./web-api-protocols/04-grpc-dotnet.md)
- [ ] Что такое GraphQL? Resolver, N+1 в GraphQL, DataLoader — [web-api-protocols/05-graphql-schema.md](./web-api-protocols/05-graphql-schema.md)
- [ ] Сравнение REST / gRPC / GraphQL — когда что выбирать? — [web-api-protocols/08-comparison.md](./web-api-protocols/08-comparison.md)

---

## 8. SOLID и принципы проектирования

- [ ] SRP — Single Responsibility Principle: что значит «одна ответственность»? Как определить границу?
- [ ] OCP — Open/Closed Principle: как расширять поведение не меняя код? Примеры через полиморфизм
- [ ] LSP — Liskov Substitution Principle: что нарушает LSP? Примеры нарушений
- [ ] ISP — Interface Segregation Principle: почему маленькие интерфейсы лучше fat interface?
- [ ] DIP — Dependency Inversion Principle: чем отличается от DI-контейнера?
- [ ] DRY, YAGNI, KISS — когда DRY вреден? Что такое wrong abstraction?
- [ ] Composition over inheritance — почему предпочтительнее? Примеры

---

## 9. Design Patterns

### Порождающие
- [ ] Singleton: как реализовать thread-safe в .NET? Почему Singleton в DI лучше?
- [ ] Factory Method vs Abstract Factory — в чём разница? Примеры применения
- [ ] Builder — когда нужен? Примеры с Fluent API

### Структурные
- [ ] Decorator — как работает? Чем отличается от Proxy? Примеры в ASP.NET (middleware)
- [ ] Adapter vs Facade — разница, примеры
- [ ] Composite — когда применять? Дерево объектов

### Поведенческие
- [ ] Strategy — как заменяет if/else? Пример с DI
- [ ] Observer — EventHandler vs IObserver, Rx Extensions
- [ ] Command — связь с CQRS. Как MediatR реализует паттерн Command
- [ ] Chain of Responsibility — как работает? Middleware как пример в ASP.NET Core

---

## 10. DDD и архитектурные паттерны

### Основы DDD
- [ ] Что такое DDD? Bounded Context, Ubiquitous Language
- [ ] Entity vs Value Object — в чём разница? Как определить?
- [ ] Что такое Aggregate и Aggregate Root? Правила консистентности внутри агрегата
- [ ] Что такое Domain Event? Как публиковать и обрабатывать?
- [ ] Что такое Repository в DDD? Чем отличается от Data Access Layer?
- [ ] Что такое Application Service vs Domain Service?

### CQRS и Event Sourcing
- [ ] Что такое CQRS? Зачем разделять модели чтения и записи? Когда оправдано?
- [ ] Как реализовать CQRS с MediatR в ASP.NET Core? IRequest, IRequestHandler
- [ ] Что такое Event Sourcing? Чем отличается от state-based хранения?
- [ ] Что такое Outbox Pattern? Зачем нужен при работе с доменными событиями и внешними системами?

### Clean Architecture
- [ ] Слои Clean Architecture / Onion Architecture: Domain, Application, Infrastructure, Presentation
- [ ] Vertical Slice Architecture — чем отличается от слоёв? Когда предпочтительнее?
- [ ] Dependency Rule — почему зависимости должны идти внутрь?

---

## 11. Безопасность и Аутентификация

- [ ] Что такое Authentication vs Authorization?
- [ ] Как работает JWT? Header, Payload, Signature. Как верифицировать токен?
- [ ] Access Token vs Refresh Token — зачем нужны оба? Как реализовать rotation?
- [ ] Как отозвать JWT-токен? (token blocklist, short TTL + refresh, versioning)
- [ ] Что такое OAuth 2.0? Flows: Authorization Code, Client Credentials, Device Code
- [ ] Что такое OpenID Connect? Чем отличается от OAuth 2.0?
- [ ] Как настроить Auth в ASP.NET Core? `AddAuthentication`, `AddJwtBearer`, `[Authorize]`
- [ ] Что такое Policy-based authorization? Claims vs Roles — когда что использовать?
- [ ] Как защититься от CSRF, XSS, SQL Injection, Mass Assignment?
- [ ] Что такое CORS? Как настроить для SPA? Preflight requests

---

## 12. Кэширование

- [ ] Виды кэширования: In-Memory, Distributed (Redis), Response Cache, Output Cache
- [ ] Что такое `IMemoryCache` vs `IDistributedCache`? Когда использовать каждый?
- [ ] Как работает Redis? Структуры данных (String, Hash, List, Set, Sorted Set)
- [ ] Cache-aside vs Write-through vs Write-behind — паттерны инвалидации
- [ ] Что такое cache stampede? Как защититься? (mutex lock, probabilistic early expiry)
- [ ] `HybridCache` (.NET 9) — что это и как работает?
- [ ] HTTP кэширование: Cache-Control, ETag, Last-Modified, conditional requests

---

## 13. Тестирование

- [ ] Unit vs Integration vs E2E тесты — пирамида тестирования. Где баланс?
- [ ] Что такое mocking? `Mock<T>` в Moq — как настроить `Setup`, `Verify`?
- [ ] Зачем использовать `Testcontainers`? Как написать интеграционный тест с реальной БД?
- [ ] Как тестировать ASP.NET Core контроллеры? `WebApplicationFactory<T>`, `HttpClient`
- [ ] Что такое Test Doubles: mock, stub, fake, spy, dummy — разница?
- [ ] Как тестировать async-код? `async Task` тесты, `await Assert.ThrowsAsync`
- [ ] Что такое Arrange-Act-Assert (AAA) паттерн?
- [ ] Как тестировать EF Core? `InMemory` provider vs Testcontainers — почему второй лучше?

---

## 14. Производительность и профилирование

- [ ] Как профилировать .NET приложение? dotnet-trace, dotnet-counters, PerfView
- [ ] Что такое `Span<T>` и `Memory<T>`? Как они избегают аллокаций?
- [ ] `ArrayPool<T>` и `MemoryPool<T>` — зачем нужны?
- [ ] Что такое `System.IO.Pipelines`? Когда использовать?
- [ ] Как избежать Garbage Pressure? Паттерны zero-allocation
- [ ] Что такое BenchmarkDotNet? Как правильно измерить производительность?
- [ ] HTTP/2 vs HTTP/3: что нового? Как включить в Kestrel?

---

## 15. Микросервисы и Messaging

- [ ] Монолит vs Микросервисы — когда переходить? Трейдоффы
- [ ] Что такое API Gateway? Задачи: routing, rate limiting, auth, aggregation
- [ ] Что такое Service Discovery? Consul, Kubernetes DNS
- [ ] Паттерны межсервисной коммуникации: sync (HTTP/gRPC) vs async (messaging)
- [ ] Что такое RabbitMQ? Exchange types (direct, fanout, topic, headers). Exactly-once delivery?
- [ ] Что такое Kafka? Topic, Partition, Consumer Group. Чем отличается от RabbitMQ?
- [ ] Что такое Saga Pattern? Choreography vs Orchestration
- [ ] Что такое Circuit Breaker? Паттерн Bulkhead. Polly в .NET
- [ ] Что такое Outbox Pattern? Как гарантировать доставку сообщений?

---

## 16. Docker и Kubernetes

- [ ] Что такое Docker? Чем контейнер отличается от VM?
- [ ] Как написать многоэтапный `Dockerfile` для .NET? Зачем multi-stage build?
- [ ] Что такое Docker Compose? Как поднять приложение с БД и Redis?
- [ ] Что такое Kubernetes? Pod, Deployment, Service, Ingress — объясни связь
- [ ] Liveness vs Readiness vs Startup probe — в чём разница? Как настроить?
- [ ] Что такое ConfigMap и Secret в Kubernetes?
- [ ] Rolling update vs Blue-Green vs Canary deployment — когда что применять?

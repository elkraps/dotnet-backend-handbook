# dotnet-handbook — структурированные конспекты по .NET для подготовки к собеседованию

<div align="center">

> Чтобы не пропустить обновления: поставь ⭐ репозиторию и подпишись на [тг-канал](https://t.me/elkraps_sandbox) — общаемся про IT, бонусом собрал PDF-сборник всех тем.

[![Telegram](https://img.shields.io/badge/Telegram-@elkraps__sandbox-26A5E4?logo=telegram&logoColor=white)](https://t.me/elkraps_sandbox)
[![GitHub stars](https://img.shields.io/github/stars/elkraps/dotnet-backend-handbook?style=social)](https://github.com/elkraps/dotnet-backend-handbook)

</div>

---

Декомпозированные конспекты по темам. Каждая папка — одна тема, разбитая на файлы по одному концепту. Читаются за один присест, содержат Mermaid-диаграммы, реальные C# примеры и разбор внутреннего устройства.

---

## Темы

### Рантайм и основы

| Тема | Описание |
|------|----------|
| [CLR Fundamentals](./clr-fundamentals/README.md) | Как работает CLR, JIT, типы данных, стек и куча, GC, финализация, интерфейсы. Фундамент для всех остальных тем. |
| [Async/Await](./async-await/README.md) | State machine под капотом `async/await`, ThreadPool, Task, SynchronizationContext, ValueTask, CancellationToken, антипаттерны. |
| [Synchronization Primitives](./synchronization-primitives/README.md) | Memory model, `lock`, Monitor, `SemaphoreSlim`, `ReaderWriterLockSlim`, Interlocked, volatile, lock-free алгоритмы, concurrent коллекции. |
| [TPL & Parallel](./tpl-parallel/README.md) | `Task.Run`, `Parallel.For`, PLINQ, TPL Dataflow, партиционирование, паттерны конвейеров и MapReduce. |

### Данные и доступ к БД

| Тема | Описание |
|------|----------|
| [EF Core + LINQ](./ef-linq-data-access/00-README.md) | IQueryable vs IEnumerable, Change Tracker, LINQ→SQL трансляция, N+1, Include/Select, связи, Fluent API, Dapper. |
| [Database Internals](./database-internals/00-README.md) | PostgreSQL под капотом: MVCC, транзакции, изоляция, блокировки, B-tree/GIN индексы, JOIN алгоритмы, шардирование, репликация, HA. |

### Веб и API

| Тема | Описание |
|------|----------|
| [ASP.NET Core Pipeline](./aspnet-pipeline/README.md) | Kestrel, жизненный цикл запроса, middleware, routing, model binding, DI, конфигурация, фоновые сервисы, фильтры. |
| [Web API Protocols](./web-api-protocols/00-README.md) | REST, gRPC/Protobuf, GraphQL/HotChocolate, OData — принципы, .NET реализация, сравнение и дерево выбора. |

### Проектирование

| Тема | Описание |
|------|----------|
| [ООП, SOLID, GRASP, паттерны](./oop-and-design/README.md) | 4 принципа ООП, SOLID, GRASP, GoF-паттерны (порождающие/структурные/поведенческие), коллекции .NET, антипаттерны. |

### Архитектура распределённых систем

| Тема | Описание |
|------|----------|
| [Микросервисы](./microservices/README.md) | Архитектурные парадигмы, коммуникация, паттерны данных (CQRS/Event Sourcing), reliability (Polly), Saga/Outbox, Gateway/Discovery, observability, deployment, антипаттерны. |

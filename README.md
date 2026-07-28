# dotnet-handbook — структурированные конспекты по .NET для подготовки к собеседованию

<div align="center">

> Собираю большой учебный репозиторий для backend разработчика и комьюнити вокруг него — конспекты, вакансии, курсы и полезные видео, все в одном месте.
> 
> Поставь ⭐ репозиторию и подпишись на [тг-канал](https://t.me/elkraps_sandbox), чтобы не пропустить обновления.

[![Telegram](https://img.shields.io/badge/Telegram-@elkraps__sandbox-26A5E4?logo=telegram&logoColor=white)](https://t.me/elkraps_sandbox)
[![GitHub stars](https://img.shields.io/github/stars/elkraps/dotnet-backend-handbook?style=social)](https://github.com/elkraps/dotnet-backend-handbook)

</div>

---

### Данный проект будет содержать или уже содержит: 
- Конспекты для подготовки к .NET собеседованию, либо к его глубокому изучению. Каждая папка — одна тема, разбитая на файлы по одному концепту. Читаются за один присест, содержат Mermaid-диаграммы, реальные C# примеры и разбор внутреннего устройства.
- Полезные видео-материалы.
- Бесплатные курсы по углубленному изучению тем, необходимых backend разработчику.
- Книги по IT.
- Полезные статьи и другие материалы.
- Компании и их вакансии для .NET разработчиков.

### В планах так же добавить:
- Конспекты по python, golang
- Skill для проверки и доработки резюме

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

## Полезные материалы

| Раздел | Описание |
|--------|----------|
| [Вакансии](./resources/vacancies.md) | Ссылки на вакансии, которые показались интересными |
| [Видео](./resources/videos.md) | Полезные видео из процесса обучения |
| [Курсы](./resources/courses.md) | Ссылки на курсы |
| [Книги](./resources/books.md) | Полезные книги по .NET и смежным темам |
| [Статьи](./resources/articles.md) | Полезные статьи и другие материалы |

## Как внести вклад

Репозиторий открыт для дополнений: нашёл полезную вакансию, видео, курс, книгу, статью или хочешь поправить/дополнить конспект — открывай PR. Для ссылок в `resources/` добавляй строку по образцу существующих в таблице. Шаблон PR с чек-листом подставится автоматически.

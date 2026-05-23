# Пайплайн выполнения async-метода

> Понять, где именно работает код — на каком потоке, в какой момент — ключ к правильному написанию async-кода.

## Содержание
- [Полный путь от вызова до результата](#полный-путь)
- [Hot path: синхронное завершение](#hot-path)
- [Cold path: асинхронное завершение](#cold-path)
- [ExecutionContext](#executioncontext)
- [AsyncLocal\<T\>](#asynclocalt)
- [Task.Yield()](#taskyield)
- [Task.Run и SynchronizationContext](#taskrun-и-synchronizationcontext)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

---

## Полный путь

Рассмотрим `await ReadFileAsync()` из кода на UI-потоке.

```mermaid
sequenceDiagram
    participant UI as UI Thread
    participant SM as StateMachine
    participant OS as OS / Hardware
    participant IOCP as I/O Completion Port
    participant TP as ThreadPool Worker

    Note over UI: var result = await ReadFileAsync()
    UI->>SM: Создать SM, builder.Start()
    SM->>SM: MoveNext() [state=-1]
    SM->>OS: FileStream.ReadAsync() → ОС начинает DMA-чтение

    Note over SM: awaiter.IsCompleted == false
    SM->>SM: state=0, подписать continuation
    SM-->>UI: return Task (incomplete)
    Note over UI: Поток UI СВОБОДЕН — обрабатывает события

    Note over OS: Диск завершил чтение (DMA)
    OS->>IOCP: Сигнал завершения I/O
    IOCP->>TP: Вызвать callback

    TP->>SM: MoveNext() [state=0]
    Note over SM: GetResult() → byte[]

    alt SynchronizationContext != null (UI-приложение)
        SM->>UI: SyncContext.Post(continuation)
        UI->>UI: Продолжить выполнение с результатом
    else SynchronizationContext == null (ASP.NET Core, Console)
        TP->>TP: Продолжить выполнение на потоке пула
    end
```

Ключевой момент: между «return Task» и «MoveNext() [state=0]» **ни один поток не занят ожиданием**. ОС уведомит, когда данные готовы. Это «true async» — принципиальное отличие от `Task.Run(() => file.Read())`, где поток пула **блокируется**.

---

## Hot path

Когда `IsCompleted == true` до подписки continuation — поток не переключается.

```mermaid
flowchart LR
    subgraph "Hot Path"
        A1["MoveNext()"] --> A2["IsCompleted → true"]
        A2 --> A3["GetResult()"]
        A3 --> A4["SetResult()"]
        A4 --> A5["Кешированный Task"]
        style A5 fill:#2d5,color:#fff
    end

    subgraph "Cold Path"
        B1["MoveNext()"] --> B2["IsCompleted → false"]
        B2 --> B3["Боксинг SM → heap"]
        B3 --> B4["Подписка continuation"]
        B4 --> B5["return, поток свободен"]
        B5 -.-> B6["...callback..."]
        B6 --> B7["Новый MoveNext()"]
        style B5 fill:#d52,color:#fff
    end
```

**Когда Task завершается синхронно:**
- `Stream.ReadAsync()` — данные уже в буфере ОС
- `Socket.ReceiveAsync()` — данные уже пришли в буфер сокета
- `MemoryCache.GetOrCreateAsync()` — попадание в кеш
- `Channel.ReadAsync()` — в канале уже есть элемент
- `SemaphoreSlim.WaitAsync()` — семафор не занят

В высоконагруженных серверах большинство async-вызовов завершаются синхронно.

**Стоимость:**

| | Hot Path | Cold Path |
|---|---|---|
| Аллокации | 0 | ~2 (SM box + Task) |
| Переключения потоков | 0 | 1+ |
| Захват контекста | нет | ExecutionContext + возможно SyncContext |
| Overhead | ~наносекунды | ~микросекунды |

---

## Cold path

Когда `IsCompleted == false`:

```mermaid
flowchart TD
    A["awaiter.IsCompleted == false"] --> B["builder.AwaitUnsafeOnCompleted(awaiter, sm)"]
    B --> C["ExecutionContext.Capture()"]
    C --> D["Создать AsyncStateMachineBox на куче\n(Task + SM + EC в одном объекте)"]
    D --> F["awaiter.UnsafeOnCompleted(box.MoveNextAction)"]
    F --> G["return — поток свободен"]

    G -.->|"I/O завершился"| H["callback → в очередь"]
    H --> I{"SynchronizationContext?"}
    I -->|"есть"| J["SyncContext.Post(MoveNextAction)"]
    I -->|"null"| K["ThreadPool.QueueUserWorkItem(MoveNextAction)"]
    J --> L["ExecutionContext.Run(captured, MoveNext)"]
    K --> L
    L --> M["SM.MoveNext() с восстановленным контекстом"]
```

**Порядок:**
1. `ExecutionContext.Capture()` — всегда, отключить нельзя
2. `AsyncStateMachineBox` — создаётся один раз при первом async await, переиспользуется для всех последующих `await`-ов в том же методе
3. Маршалинг continuation: через SyncContext или ThreadPool

---

## ExecutionContext

«Папка с документами», которая течёт вместе с кодом через `await`. Невидимый контейнер.

**Что внутри:**
- `AsyncLocal<T>` — основной механизм хранения ambient-данных
- SecurityContext (в .NET Framework)

**Отличие от SynchronizationContext:**

| | ExecutionContext | SynchronizationContext |
|---|---|---|
| **Что** | Данные (AsyncLocal, Security) | Целевой «планировщик» |
| **Зачем** | Пронести данные через async | Вернуться на нужный поток |
| **Кто управляет** | Всегда захватывается builder'ом | Захватывается awaiter'ом |
| **Можно отключить?** | Нет | Да, через `ConfigureAwait(false)` |

`ExecutionContext` захватывается **всегда**, независимо от `ConfigureAwait(false)`. Это намеренно — иначе `AsyncLocal<T>` и security данные терялись бы через `await`.

---

## AsyncLocal\<T\>

Хранит данные в `ExecutionContext` с **copy-on-write** семантикой.

```csharp
var local = new AsyncLocal<string>();

async Task Parent()
{
    local.Value = "parent";
    await Child();
    Console.WriteLine(local.Value); // "parent" — не "child"!
}

async Task Child()
{
    Console.WriteLine(local.Value); // "parent" — унаследовали
    local.Value = "child";          // создаётся КОПИЯ EC
    Console.WriteLine(local.Value); // "child"
}
// Изменение в Child не видно в Parent — copy-on-write
```

Это принципиальное отличие от `ThreadLocal<T>`, который:
- Привязан к конкретному потоку
- Не течёт через `await` (поток может смениться)

**Типичный use case — correlation ID:**

```csharp
private static readonly AsyncLocal<string> requestId = new();

public async Task Handle()
{
    requestId.Value = Guid.NewGuid().ToString();
    await DoWork();
    // requestId.Value здесь тот же — EC восстановлен
}

public async Task DoWork()
{
    // Работает даже если мы на другом потоке после await
    logger.LogInformation("Processing {RequestId}", requestId.Value);
}
```

---

## Task.Yield()

`await Task.Yield()` **всегда** форсирует асинхронный путь. У `YieldAwaitable` `IsCompleted` всегда `false`.

```csharp
public async Task Process(IEnumerable<Item> items)
{
    foreach (var item in items)
    {
        Handle(item);
        await Task.Yield(); // после каждой итерации — отдать поток пулу
        // Другие work items получают шанс выполниться
    }
}
```

**Когда использовать:**
- Длинные синхронные циклы, чтобы не монополизировать поток
- Принудительный переход на ThreadPool
- Тестирование: форсировать async-поведение для проверки race conditions

**Под капотом:** `YieldAwaitable.GetAwaiter().IsCompleted` возвращает `false`. При подписке continuation — маршалит через текущий `SyncContext` (если есть) или `ThreadPool`.

---

## Task.Run и SynchronizationContext

`Task.Run()` **специально убирает** `SynchronizationContext` внутри делегата:

```csharp
// На UI-потоке (SyncContext = DispatcherSynchronizationContext):
await Task.Run(async () =>
{
    // Здесь SynchronizationContext.Current == null!
    // Task.Run вызывает SetSynchronizationContext(null) перед выполнением
    await SomeAsync(); // нет риска deadlock'а
});
```

`Task.Factory.StartNew()` **не убирает** SyncContext — это одна из причин предпочитать `Task.Run()`:

```csharp
// Task.Factory.StartNew — подвох:
await Task.Factory.StartNew(async () =>
{
    await SomeAsync(); // SyncContext может быть != null и вызвать deadlock
}, CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
```

---

## Подводные камни

**Код до первого await выполняется на потоке вызывающего.** Если до первого `await` есть тяжёлые вычисления — они блокируют вызывающий поток (UI или ASP.NET request thread).

**`await` не гарантирует переключение потока.** Если Task уже завершён (`IsCompleted == true`), код продолжится на том же потоке — без переключения. Хочешь гарантированного переключения — используй `await Task.Yield()` или `await Task.Run(...)`.

**После `await` поток может смениться.** Переменные, захваченные в замыканиях, могут использоваться из другого потока. `ThreadLocal<T>` после `await` может вернуть другое значение.

---

## См. также

- [01-threadpool.md](./01-threadpool.md) — где выполняются continuation'ы
- [03-state-machine.md](./03-state-machine.md) — как MoveNext управляет переходами
- [06-synchronization-context.md](./06-synchronization-context.md) — как SyncContext влияет на маршалинг

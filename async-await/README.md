# Async/Await в .NET

`async/await` — синтаксический сахар поверх конечного автомата (state machine). Компилятор трансформирует линейный код в struct с методом `MoveNext()`. Рантайм не знает об `async` — он работает с ThreadPool, Task и callback'ами. Этот раздел разбирает всю механику от уровня ОС до антипаттернов.

## Порядок чтения

Если читаешь первый раз — иди по порядку. Каждый файл опирается на предыдущий.

1. [01-threadpool.md](./01-threadpool.md) — ThreadPool: архитектура очередей, work stealing, IOCP, Hill Climbing, thread starvation
2. [02-task.md](./02-task.md) — Task: два способа создания, состояния, что хранит внутри, исключения
3. [03-state-machine.md](./03-state-machine.md) — Трансформация компилятором: IAsyncStateMachine, MoveNext, боксинг на heap
4. [04-awaitable-pattern.md](./04-awaitable-pattern.md) — Паттерн awaitable/awaiter: duck typing, AsyncTaskMethodBuilder, кастомные awaiter'ы
5. [05-execution-flow.md](./05-execution-flow.md) — Полный пайплайн: hot path vs cold path, ExecutionContext, AsyncLocal, Task.Yield
6. [06-synchronization-context.md](./06-synchronization-context.md) — SynchronizationContext и ConfigureAwait: захват контекста, deadlock, правила применения
7. [07-valuetask.md](./07-valuetask.md) — ValueTask: аллокации, IValueTaskSource, пулинг state machine
8. [08-cancellation.md](./08-cancellation.md) — CancellationToken: cooperative cancellation, linked tokens, callback
9. [09-advanced.md](./09-advanced.md) — WhenAll/WhenAny, IAsyncEnumerable, Channel\<T\>, TaskCompletionSource
10. [10-antipatterns.md](./10-antipatterns.md) — Антипаттерны: async void, deadlock, fire-and-forget, async/sync over sync/async

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| ThreadPool, work stealing | [01-threadpool.md](./01-threadpool.md) |
| Thread starvation | [01-threadpool.md](./01-threadpool.md#thread-starvation) |
| Task состояния, жизненный цикл | [02-task.md](./02-task.md) |
| await vs .Result (исключения) | [02-task.md](./02-task.md#исключения-await-vs-result) |
| IAsyncStateMachine, MoveNext | [03-state-machine.md](./03-state-machine.md) |
| Боксинг struct → heap | [03-state-machine.md](./03-state-machine.md#боксинг-на-heap) |
| Код до первого await | [03-state-machine.md](./03-state-machine.md#код-до-первого-await) |
| GetAwaiter(), IsCompleted, OnCompleted | [04-awaitable-pattern.md](./04-awaitable-pattern.md) |
| AsyncTaskMethodBuilder | [04-awaitable-pattern.md](./04-awaitable-pattern.md#asynctaskmethodbuilder) |
| Hot path vs cold path | [05-execution-flow.md](./05-execution-flow.md) |
| ExecutionContext | [05-execution-flow.md](./05-execution-flow.md#executioncontext) |
| AsyncLocal\<T\> | [05-execution-flow.md](./05-execution-flow.md#asynclocalt) |
| Task.Yield() | [05-execution-flow.md](./05-execution-flow.md#taskyield) |
| SynchronizationContext | [06-synchronization-context.md](./06-synchronization-context.md) |
| ConfigureAwait(false) | [06-synchronization-context.md](./06-synchronization-context.md#configureawaitfalse) |
| Deadlock механизм | [06-synchronization-context.md](./06-synchronization-context.md#deadlock) |
| ValueTask vs Task | [07-valuetask.md](./07-valuetask.md) |
| IValueTaskSource, пулинг | [07-valuetask.md](./07-valuetask.md#ivaluetasksource-и-пулинг) |
| CancellationToken | [08-cancellation.md](./08-cancellation.md) |
| Linked tokens, timeout | [08-cancellation.md](./08-cancellation.md#linked-tokens) |
| Task.WhenAll / WhenAny | [09-advanced.md](./09-advanced.md) |
| IAsyncEnumerable\<T\> | [09-advanced.md](./09-advanced.md#iasyncenumerablet) |
| Channel\<T\> | [09-advanced.md](./09-advanced.md#channelt) |
| TaskCompletionSource | [09-advanced.md](./09-advanced.md#taskcompletionsource) |
| async void | [10-antipatterns.md](./10-antipatterns.md#async-void) |
| Fire-and-forget | [10-antipatterns.md](./10-antipatterns.md#fire-and-forget) |
| Async over sync / sync over async | [10-antipatterns.md](./10-antipatterns.md#async-over-sync) |

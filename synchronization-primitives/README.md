# Примитивы синхронизации

Конспекты по синхронизации в .NET: от memory model и фундаментальных проблем до lock-free алгоритмов, async-совместимых примитивов и concurrent коллекций. Тема охватывает всё — от `lock` (20 нс) до `Mutex` (1-2 мкс) и объясняет, когда что выбирать.

## Порядок чтения

1. [01-fundamentals.md](./01-fundamentals.md) — Race condition vs data race, критическая секция, atomicity / visibility / ordering, memory model .NET
2. [02-lock-monitor.md](./02-lock-monitor.md) — lock изнутри (thin lock → fat lock), Monitor.Wait/Pulse, SpinLock, SpinWait
3. [03-rw-sem-event.md](./03-rw-sem-event.md) — ReaderWriterLockSlim, SemaphoreSlim, ManualResetEventSlim, AutoResetEvent
4. [04-kernel-mode.md](./04-kernel-mode.md) — Mutex, Semaphore, EventWaitHandle, стоимость kernel-mode vs user-mode
5. [05-interlocked-volatile.md](./05-interlocked-volatile.md) — Interlocked (CAS), volatile, Volatile.Read/Write, lock-free алгоритмы
6. [06-async-sync.md](./06-async-sync.md) — SemaphoreSlim.WaitAsync, почему lock нельзя с await, AsyncLock, AsyncLocal\<T\>
7. [07-concurrent-collections.md](./07-concurrent-collections.md) — ConcurrentDictionary, ConcurrentQueue/Stack/Bag, BlockingCollection, Immutable
8. [08-problems.md](./08-problems.md) — Deadlock, priority inversion, lock convoy, async deadlock, over-synchronization

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Race condition vs data race | [01-fundamentals.md](./01-fundamentals.md) |
| Atomicity, visibility, ordering | [01-fundamentals.md](./01-fundamentals.md) |
| Memory model .NET, volatile семантика | [01-fundamentals.md](./01-fundamentals.md) |
| lock изнутри (thin lock, fat lock) | [02-lock-monitor.md](./02-lock-monitor.md) |
| Object header, sync block table | [02-lock-monitor.md](./02-lock-monitor.md) |
| Monitor.Wait / Pulse / PulseAll | [02-lock-monitor.md](./02-lock-monitor.md) |
| SpinLock, SpinWait | [02-lock-monitor.md](./02-lock-monitor.md) |
| ReaderWriterLockSlim (read-heavy) | [03-rw-sem-event.md](./03-rw-sem-event.md) |
| SemaphoreSlim (throttling, N concurrent) | [03-rw-sem-event.md](./03-rw-sem-event.md) |
| ManualResetEventSlim, AutoResetEvent | [03-rw-sem-event.md](./03-rw-sem-event.md) |
| Mutex (single-instance, cross-process) | [04-kernel-mode.md](./04-kernel-mode.md) |
| Kernel-mode vs user-mode стоимость | [04-kernel-mode.md](./04-kernel-mode.md) |
| Interlocked.CompareExchange (CAS) | [05-interlocked-volatile.md](./05-interlocked-volatile.md) |
| volatile keyword | [05-interlocked-volatile.md](./05-interlocked-volatile.md) |
| Lock-free алгоритмы (CAS loop) | [05-interlocked-volatile.md](./05-interlocked-volatile.md) |
| Почему lock нельзя с await | [06-async-sync.md](./06-async-sync.md) |
| SemaphoreSlim.WaitAsync | [06-async-sync.md](./06-async-sync.md) |
| AsyncLock паттерн | [06-async-sync.md](./06-async-sync.md) |
| AsyncLocal\<T\> vs ThreadLocal\<T\> | [06-async-sync.md](./06-async-sync.md) |
| ConcurrentDictionary (lock striping) | [07-concurrent-collections.md](./07-concurrent-collections.md) |
| ConcurrentQueue/Stack (lock-free) | [07-concurrent-collections.md](./07-concurrent-collections.md) |
| BlockingCollection vs Channel\<T\> | [07-concurrent-collections.md](./07-concurrent-collections.md) |
| Immutable коллекции + Interlocked | [07-concurrent-collections.md](./07-concurrent-collections.md) |
| Deadlock (порядок захвата, lock ordering) | [08-problems.md](./08-problems.md) |
| Lock convoy | [08-problems.md](./08-problems.md) |
| Async deadlock (.Result/.Wait()) | [08-problems.md](./08-problems.md) |
| Over-synchronization антипаттерны | [08-problems.md](./08-problems.md) |

## Алгоритм выбора примитива

```
Нужна cross-process синхронизация?
├── Да → Mutex / Semaphore / EventWaitHandle
└── Нет → Async контекст (await)?
    ├── Да → SemaphoreSlim.WaitAsync / Channel<T> / Nito.AsyncEx
    └── Нет → Паттерн?
        ├── Mutual exclusion → lock (Monitor)
        ├── N concurrent → SemaphoreSlim
        ├── Read-heavy (10:1+) → ReaderWriterLockSlim
        ├── Atomic counter → Interlocked
        ├── Simple flag → volatile
        ├── Очень короткая секция → SpinLock
        ├── Сигнал всем → ManualResetEventSlim
        ├── Сигнал одному → SemaphoreSlim(0,1)
        ├── Producer-consumer (sync) → BlockingCollection<T>
        └── Thread-safe коллекция → ConcurrentDictionary / ConcurrentQueue
```


# CLR Fundamentals

Основы рантайма .NET: как работает CLR, JIT, типы данных, память, GC и интерфейсы. Читать перед любой темой о производительности, многопоточности или архитектуре — всё остальное строится на этом фундаменте.

## Порядок чтения

1. [01-dotnet-platform.md](./01-dotnet-platform.md) — CLR, JIT (Tiered Compilation, PGO), NativeAOT
2. [02-value-reference-types.md](./02-value-reference-types.md) — Value vs Reference: копирование, equality, где хранятся данные
3. [03-string.md](./03-string.md) — string: иммутабельность, interning, StringBuilder, Span
4. [04-stack-heap.md](./04-stack-heap.md) — Stack frames, анатомия объекта на heap, стоимость аллокации
5. [05-struct-class-record.md](./05-struct-class-record.md) — struct vs class, readonly struct, ref struct, records
6. [06-boxing.md](./06-boxing.md) — Механика boxing/unboxing, неявный boxing, как избежать
7. [07-gc.md](./07-gc.md) — Поколения, фазы сборки, GC Roots, memory leaks, LOH/POH, диагностика
8. [08-finalization-dispose.md](./08-finalization-dispose.md) — Финализаторы, IDisposable паттерн, SafeHandle
9. [09-interfaces.md](./09-interfaces.md) — Контракт vs реализация, default methods, covariance/contravariance

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| CLR, JIT, RyuJIT | [01-dotnet-platform.md](./01-dotnet-platform.md) |
| Tiered Compilation, PGO | [01-dotnet-platform.md](./01-dotnet-platform.md) |
| NativeAOT vs JIT | [01-dotnet-platform.md](./01-dotnet-platform.md) |
| Value type vs Reference type | [02-value-reference-types.md](./02-value-reference-types.md) |
| Где хранятся данные (stack/heap) | [02-value-reference-types.md](./02-value-reference-types.md) |
| Nullable\<T\>, enum | [02-value-reference-types.md](./02-value-reference-types.md) |
| decimal vs double | [02-value-reference-types.md](./02-value-reference-types.md) |
| String interning | [03-string.md](./03-string.md) |
| StringBuilder | [03-string.md](./03-string.md) |
| Span\<char\>, zero-allocation | [03-string.md](./03-string.md) |
| Stack frame | [04-stack-heap.md](./04-stack-heap.md) |
| Object header (Sync Block + MT*) | [04-stack-heap.md](./04-stack-heap.md) |
| Стоимость аллокации | [04-stack-heap.md](./04-stack-heap.md) |
| struct vs class | [05-struct-class-record.md](./05-struct-class-record.md) |
| readonly struct, ref struct | [05-struct-class-record.md](./05-struct-class-record.md) |
| record class, record struct | [05-struct-class-record.md](./05-struct-class-record.md) |
| Boxing/Unboxing механика | [06-boxing.md](./06-boxing.md) |
| Неявный boxing | [06-boxing.md](./06-boxing.md) |
| Constrained call | [06-boxing.md](./06-boxing.md) |
| GC поколения Gen0/1/2 | [07-gc.md](./07-gc.md) |
| Mark / Sweep / Compact | [07-gc.md](./07-gc.md) |
| GC Roots, memory leaks | [07-gc.md](./07-gc.md) |
| LOH, POH, ArrayPool | [07-gc.md](./07-gc.md) |
| Workstation vs Server GC | [07-gc.md](./07-gc.md) |
| IDisposable паттерн | [08-finalization-dispose.md](./08-finalization-dispose.md) |
| using, await using | [08-finalization-dispose.md](./08-finalization-dispose.md) |
| SafeHandle | [08-finalization-dispose.md](./08-finalization-dispose.md) |
| Interface vs Abstract Class | [09-interfaces.md](./09-interfaces.md) |
| Default Interface Methods | [09-interfaces.md](./09-interfaces.md) |
| Explicit Implementation | [09-interfaces.md](./09-interfaces.md) |
| Covariance / Contravariance | [09-interfaces.md](./09-interfaces.md) |

## Источники

- `ИИ конспекты/clr-fundamentals.md`

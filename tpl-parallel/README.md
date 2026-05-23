# TPL & Parallel

Набор конспектов по параллелизму в .NET: от низкоуровневого Task.Run до высокоуровневых паттернов конвейеров и MapReduce. Охватывает TPL, PLINQ, TPL Dataflow, Partitioner и типичные проблемы.

## Порядок чтения

1. [01-concurrency-vs-parallelism.md](./01-concurrency-vs-parallelism.md) — Concurrency vs Parallelism vs Asynchrony: разница и выбор инструмента
2. [02-task-run.md](./02-task-run.md) — Task.Run изнутри, vs Task.Factory.StartNew, TaskCreationOptions, ContinueWith
3. [03-parallel.md](./03-parallel.md) — Parallel.For/ForEach/ForEachAsync/Invoke, thread-local state, partitioning
4. [04-plinq.md](./04-plinq.md) — AsParallel, WithDegreeOfParallelism, AsOrdered, ForAll, merge options, MapReduce
5. [05-dataflow.md](./05-dataflow.md) — TPL Dataflow: блоки, linking, BoundedCapacity, routing, pipeline
6. [06-partitioning.md](./06-partitioning.md) — Work stealing, Range vs Chunk, Partitioner\<T\>, кастомный Partitioner
7. [07-problems.md](./07-problems.md) — False sharing, thread starvation, overhead параллелизма, CPU vs I/O правило
8. [08-patterns.md](./08-patterns.md) — Producer/Consumer, MapReduce, Channel pipeline, Batch+Parallel, throttling

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Когда нужен параллелизм vs async | [01-concurrency-vs-parallelism.md](./01-concurrency-vs-parallelism.md) |
| Task.Run vs Task.Factory.StartNew | [02-task-run.md](./02-task-run.md) |
| LongRunning, DenyChildAttach | [02-task-run.md](./02-task-run.md) |
| Parallel.ForEach + thread-local | [03-parallel.md](./03-parallel.md) |
| Parallel.ForEachAsync (.NET 6+) | [03-parallel.md](./03-parallel.md) |
| PLINQ AsParallel, AsOrdered | [04-plinq.md](./04-plinq.md) |
| PLINQ Aggregate (MapReduce) | [04-plinq.md](./04-plinq.md) |
| TPL Dataflow блоки и linking | [05-dataflow.md](./05-dataflow.md) |
| Backpressure через BoundedCapacity | [05-dataflow.md](./05-dataflow.md) |
| Range vs Chunk partitioning | [06-partitioning.md](./06-partitioning.md) |
| Work stealing | [06-partitioning.md](./06-partitioning.md) |
| Кастомный Partitioner по ключу | [06-partitioning.md](./06-partitioning.md) |
| False sharing | [07-problems.md](./07-problems.md) |
| Thread starvation (sync + async) | [07-problems.md](./07-problems.md) |
| CPU-bound vs I/O-bound правило | [07-problems.md](./07-problems.md) |
| Producer/Consumer + Channel | [08-patterns.md](./08-patterns.md) |
| Channel pipeline (многостадийный) | [08-patterns.md](./08-patterns.md) |
| Throttling (rate limiting) | [08-patterns.md](./08-patterns.md) |

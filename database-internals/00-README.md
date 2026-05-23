# Database Internals

Конспекты по внутреннему устройству PostgreSQL: транзакции, индексы, репликация, шардирование, оптимизация запросов. Фокус — механизмы под капотом, а не синтаксис SQL.

## Порядок чтения

1. [01-transaction-isolation.md](./01-transaction-isolation.md) — уровни изоляции, аномалии, write skew
2. [02-mvcc.md](./02-mvcc.md) — как PostgreSQL реализует изоляцию без блокировок через версионирование строк
3. [03-locking.md](./03-locking.md) — SELECT FOR UPDATE, deadlock, optimistic locking в EF Core
4. [04-indexes.md](./04-indexes.md) — B-tree internals, типы индексов, composite/covering/partial, HOT update
5. [05-join-algorithms.md](./05-join-algorithms.md) — Nested Loop / Hash Join / Merge Join, N+1 проблема
6. [06-sharding.md](./06-sharding.md) — Hash/Range/List sharding, consistent hashing, cross-shard queries, 2PC
7. [07-replication.md](./07-replication.md) — WAL, streaming vs logical replication, sync vs async
8. [08-high-availability.md](./08-high-availability.md) — Patroni, pgBouncer, Raft consensus
9. [09-query-optimization.md](./09-query-optimization.md) — EXPLAIN ANALYZE, scan types, planner, типичные проблемы

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Dirty Read, Phantom, Write Skew | [01-transaction-isolation.md](./01-transaction-isolation.md) |
| xmin, xmax, snapshot, VACUUM | [02-mvcc.md](./02-mvcc.md) |
| SELECT FOR UPDATE, SKIP LOCKED | [03-locking.md](./03-locking.md) |
| Deadlock | [03-locking.md](./03-locking.md) |
| DbUpdateConcurrencyException (EF Core) | [03-locking.md](./03-locking.md) |
| B-tree, GIN, BRIN | [04-indexes.md](./04-indexes.md) |
| Left-prefix rule, INCLUDE | [04-indexes.md](./04-indexes.md) |
| HOT update, fillfactor | [04-indexes.md](./04-indexes.md) |
| Hash Join, Merge Join, Nested Loop | [05-join-algorithms.md](./05-join-algorithms.md) |
| N+1 проблема, Include в EF Core | [05-join-algorithms.md](./05-join-algorithms.md) |
| Consistent hashing | [06-sharding.md](./06-sharding.md) |
| Two-Phase Commit (2PC), Saga | [06-sharding.md](./06-sharding.md) |
| WAL, LSN, streaming replication | [07-replication.md](./07-replication.md) |
| RPO, RTO, synchronous_commit | [07-replication.md](./07-replication.md) |
| Logical replication, CDC | [07-replication.md](./07-replication.md) |
| Patroni failover алгоритм | [08-high-availability.md](./08-high-availability.md) |
| pgBouncer pool_mode | [08-high-availability.md](./08-high-availability.md) |
| Raft, quorum, split-brain | [08-high-availability.md](./08-high-availability.md) |
| EXPLAIN ANALYZE, cost model | [09-query-optimization.md](./09-query-optimization.md) |
| Keyset pagination, OFFSET | [09-query-optimization.md](./09-query-optimization.md) |
| Partition pruning, pg_partman | [09-query-optimization.md](./09-query-optimization.md) |

## Источники

- `ИИ конспекты/databases-internals.md` — исходный конспект (~1095 строк, PostgreSQL internals)

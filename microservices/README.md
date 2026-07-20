# Микросервисы

От выбора архитектуры (монолит vs микросервисы) до продовой выкладки: коммуникация, данные, надёжность, распределённые транзакции, инфраструктура, наблюдаемость и типичные способы всё сломать.

## Порядок чтения

1. [01-architecture-paradigms.md](./01-architecture-paradigms.md) — монолит, modular monolith, микросервисы, Bounded Context, Conway's Law, миграция (Strangler Fig, ACL)
2. [02-communication.md](./02-communication.md) — REST, gRPC, Message Broker, Choreography vs Orchestration
3. [03-data-patterns.md](./03-data-patterns.md) — Database per Service, CQRS, Event Sourcing
4. [04-reliability-polly.md](./04-reliability-polly.md) — Circuit Breaker, Retry, Bulkhead, Timeout, Fallback (Polly)
5. [05-saga-outbox.md](./05-saga-outbox.md) — распределённые транзакции: Saga, Outbox, Inbox
6. [06-gateway-discovery.md](./06-gateway-discovery.md) — Service Discovery, API Gateway, BFF, Service Mesh
7. [07-observability.md](./07-observability.md) — трейсинг, логирование, метрики, health checks
8. [08-deployment.md](./08-deployment.md) — Docker, Kubernetes, стратегии деплоя, Feature Flags
9. [09-antipatterns.md](./09-antipatterns.md) — GRASP в микросервисах, Distributed Monolith, Chatty Services и другие антипаттерны

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Монолит vs Modular Monolith vs Microservices | [01-architecture-paradigms.md](./01-architecture-paradigms.md) |
| Bounded Context, Context Map, Conway's Law | [01-architecture-paradigms.md](./01-architecture-paradigms.md) |
| Strangler Fig, Anti-Corruption Layer | [01-architecture-paradigms.md](./01-architecture-paradigms.md) |
| REST-версионирование, gRPC (unary/streaming) | [02-communication.md](./02-communication.md) |
| Event Notification vs Event-Carried State Transfer | [02-communication.md](./02-communication.md) |
| Choreography vs Orchestration | [02-communication.md](./02-communication.md) |
| Database per Service | [03-data-patterns.md](./03-data-patterns.md) |
| CQRS, Eventual Consistency | [03-data-patterns.md](./03-data-patterns.md) |
| Event Sourcing, Snapshot | [03-data-patterns.md](./03-data-patterns.md) |
| Circuit Breaker (Closed/Open/Half-Open) | [04-reliability-polly.md](./04-reliability-polly.md) |
| Retry + Exponential Backoff + Jitter | [04-reliability-polly.md](./04-reliability-polly.md) |
| Bulkhead, Timeout, Fallback, Policy.WrapAsync | [04-reliability-polly.md](./04-reliability-polly.md) |
| Saga (Choreography / Orchestration) | [05-saga-outbox.md](./05-saga-outbox.md) |
| Outbox Pattern, Inbox Pattern | [05-saga-outbox.md](./05-saga-outbox.md) |
| Service Discovery (Client/Server-side) | [06-gateway-discovery.md](./06-gateway-discovery.md) |
| API Gateway (YARP), BFF | [06-gateway-discovery.md](./06-gateway-discovery.md) |
| Service Mesh (Istio, sidecar) | [06-gateway-discovery.md](./06-gateway-discovery.md) |
| Distributed Tracing (OpenTelemetry, W3C Trace Context) | [07-observability.md](./07-observability.md) |
| Structured Logging, correlation ID | [07-observability.md](./07-observability.md) |
| RED / USE методы, Health Checks | [07-observability.md](./07-observability.md) |
| Docker multi-stage, Kubernetes объекты | [08-deployment.md](./08-deployment.md) |
| Rolling Update, Blue-Green, Canary | [08-deployment.md](./08-deployment.md) |
| Feature Flags | [08-deployment.md](./08-deployment.md) |
| GRASP в микросервисах | [09-antipatterns.md](./09-antipatterns.md) |
| Distributed Monolith, Chatty Services, Big Bang Rewrite | [09-antipatterns.md](./09-antipatterns.md) |

## Итоговая шпаргалка

| Тема | Ключевые идеи |
|------|--------------|
| Монолит vs Микросервисы | Начни с Modular Monolith; микросервисы — когда команда > 10, масштабирование нужно независимо |
| Bounded Context | Граница, внутри которой модель имеет точное значение; основа декомпозиции |
| Conway's Law | Архитектура системы = структура команды; меняй команды → меняется архитектура |
| gRPC | Binary, contract-first, HTTP/2, streaming; для internal service-to-service |
| Choreography vs Orchestration | Choreography — слабая связность, сложный flow; Orchestration — явный flow, coupling |
| CQRS | Разные модели для чтения и записи; eventual consistency |
| Event Sourcing | Состояние = replay событий; полный audit trail; snapshot каждые N событий |
| Circuit Breaker | Closed→Open→Half-Open; fast-fail при недоступности downstream |
| Retry + Jitter | Exponential backoff ±jitter; только для идемпотентных операций |
| Saga | Локальные транзакции + компенсации; Choreography через события, Orchestration через координатор |
| Outbox Pattern | Атомарная запись в БД + event; Relay публикует в Broker; решает dual-write |
| Inbox Pattern | Idempotency key в БД; защита от дубликатов при at-least-once |
| API Gateway | Единая точка входа; auth, rate limiting, routing, aggregation |
| Service Mesh | Istio/Envoy sidecar; mTLS, tracing, retry — без изменения кода сервиса |
| Strangler Fig | Постепенная миграция; Façade → вынести один BC → перенаправить трафик |
| OpenTelemetry | Traces (Jaeger), Metrics (Prometheus), Logs (ELK); W3C Trace Context для propagation |
| Liveness vs Readiness | Liveness fail → restart; Readiness fail → убрать из rotation |
| Rolling vs Blue-Green vs Canary | Rolling — default; Blue-Green — мгновенный switch; Canary — постепенный % |

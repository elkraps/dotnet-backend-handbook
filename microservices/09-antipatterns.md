# GRASP в микросервисах и антипаттерны

> Те же принципы распределения ответственности, что и внутри одного класса, работают на уровне сервисов. А нарушение этих принципов на уровне архитектуры стоит дороже — это не рефакторинг класса, а перепроектирование границ между командами.

## Содержание
- [Information Expert](#information-expert)
- [Low Coupling](#low-coupling)
- [High Cohesion](#high-cohesion)
- [Indirection](#indirection)
- [Protected Variations](#protected-variations)
- [Distributed Monolith](#distributed-monolith)
- [Chatty Services](#chatty-services-болтливые-сервисы)
- [Shared Libraries с бизнес-логикой](#shared-libraries-с-бизнес-логикой)
- [No API Versioning](#no-api-versioning)
- [Big Bang Rewrite](#big-bang-rewrite)
- [См. также](#см-также)

## Information Expert

Ответственность назначается сервису, который владеет нужными данными.

```
Кто должен считать сумму заказа? → Order Service (владеет items и ценами)
Кто должен проверять кредитный лимит? → Payment Service (владеет балансом)
```

Нарушение: Order Service обращается к Payment Service за данными, чтобы посчитать что-то, что логически принадлежит Payment.

## Low Coupling

Сервисы слабо связаны — изменение одного не требует изменения другого. Достигается через event-driven коммуникацию вместо прямых HTTP-вызовов, контракты через события/схемы (а не через shared code), Database per Service (нет разделённой схемы).

## High Cohesion

Каждый сервис делает одну вещь хорошо. Функциональность, которая изменяется вместе, живёт вместе.

```
Плохо: "ProductOrderUserService" — всё в одном
Хорошо: OrderService (создание/управление), ProductService (каталог), UserService (профили)
```

## Indirection

Избегай прямой связи, используй посредников: API Gateway, Message Broker, Service Registry.

```
Client → API Gateway → Order Service      (не напрямую Client → Order Service:8080)
Order Service → Kafka → Payment Service   (не напрямую HTTP к Payment)
```

## Protected Variations

Скрой точки изменчивости за стабильным интерфейсом.

```csharp
public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(decimal amount, string currency, CancellationToken ct);
}
// Реализации: Stripe, PayPal, внутренняя система — смена провайдера не затрагивает Order Service
```

## Distributed Monolith

Самый опасный антипаттерн: выглядит как микросервисы, ведёт себя как монолит.

Признаки: сервисы можно деплоить только вместе (coupling); сервисы разделяют одну БД или схему; изменение в одном сервисе требует изменений в 5 других; нет чёткого Bounded Context — сервисы разбиты технически (Service, Repository, Controller), а не по доменным границам.

Хуже монолита: вся сложность распределённых систем без преимуществ независимого деплоя.

## Chatty Services (болтливые сервисы)

```
Один HTTP-запрос клиента → 50 синхронных межсервисных вызовов → latency >> 1 сек

Признаки: API слишком мелкозернистый
  GET /order/{id}/user → GET /user/{userId} → GET /user/{userId}/address → GET /address/{id}/...
```

Решения: агрегация в BFF (см. [Service Discovery и API Gateway](./06-gateway-discovery.md#bff-backend-for-frontend)), event-carried state transfer (см. [Коммуникация](./02-communication.md#message-broker)), CQRS read model — денормализация (см. [Паттерны данных](./03-data-patterns.md#cqrs-command-query-responsibility-segregation)).

## Shared Libraries с бизнес-логикой

```
Плохо: shared NuGet пакет "CompanyCore" содержит:
  - доменные модели (Order, Product)
  - бизнес-правила (DiscountCalculator)
  - интеграционный клиент к Auth Service

→ Изменение пакета требует обновления и редеплоя ВСЕХ сервисов
→ Это coupling через код, а не через сеть
```

Что допустимо в shared: инфраструктурный код (логирование, health checks, OpenTelemetry bootstrap), примитивы (`Result<T>`, `ValidationError`). Никаких доменных моделей и бизнес-правил.

## No API Versioning

```
Плохо:
  v1 clients: GET /orders/{id} → {id, amount, status}
  Deploy new version: GET /orders/{id} → {id, totalAmount, orderStatus}  ← BREAKING CHANGE
  → v1 clients broke
```

Версионирование нужно с первого дня, как только появляется хоть один внешний consumer (см. [Коммуникация: версионирование REST](./02-communication.md#httprest)).

## Big Bang Rewrite

```
"Давайте перепишем монолит в микросервисы за 6 месяцев"
→ 18 месяцев
→ Новая система не готова
→ Старая система всё ещё в production и продолжает развиваться
→ Нужно переносить изменения из старой в новую
→ Chaos
```

Правильный путь — Strangler Fig: постепенная миграция по одному Bounded Context за раз (см. [Архитектурные парадигмы](./01-architecture-paradigms.md#strangler-fig-pattern)).

## См. также

- [Архитектурные парадигмы](./01-architecture-paradigms.md)
- [Паттерны данных](./03-data-patterns.md) — Database per Service против Distributed Monolith
- [Коммуникация между сервисами](./02-communication.md) — Chatty Services и способы борьбы

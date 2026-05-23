# ASP.NET Core Pipeline

Полный разбор внутренностей ASP.NET Core: от байт на TCP-сокете до ответа клиенту. Покрыты Kestrel, middleware, routing, model binding, DI, конфигурация, фоновые сервисы и фильтры.

## Порядок чтения

1. [01-kestrel.md](./01-kestrel.md) — HTTP-сервер, System.IO.Pipelines, HTTP/2, TLS, reverse proxy
2. [02-request-lifecycle.md](./02-request-lifecycle.md) — полный путь запроса, HttpContext, DI Scope
3. [03-middleware.md](./03-middleware.md) — Use/Run/Map, порядок, своё middleware, convention vs IMiddleware
4. [04-routing.md](./04-routing.md) — Endpoint Routing, route templates, Minimal API, constraints
5. [05-model-binding.md](./05-model-binding.md) — источники привязки, IModelBinder, DataAnnotations, FluentValidation
6. [06-exception-handling.md](./06-exception-handling.md) — IExceptionHandler, цепочка обработчиков, Problem Details
7. [07-dependency-injection.md](./07-dependency-injection.md) — Lifetimes, Captive Dependency, Keyed Services, IOptions
8. [08-configuration.md](./08-configuration.md) — источники конфигурации, приоритеты, User Secrets, reload
9. [09-hosted-services.md](./09-hosted-services.md) — IHostedService, BackgroundService, graceful shutdown
10. [10-filters.md](./10-filters.md) — типы фильтров, порядок, ServiceFilter vs TypeFilter, сводная диаграмма

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Kestrel, HTTP/2, HTTP/3 | [01-kestrel.md](./01-kestrel.md) |
| TLS Handshake | [01-kestrel.md](./01-kestrel.md) |
| Reverse Proxy, ForwardedHeaders | [01-kestrel.md](./01-kestrel.md) |
| HttpContext — структура | [02-request-lifecycle.md](./02-request-lifecycle.md) |
| IFeatureCollection | [02-request-lifecycle.md](./02-request-lifecycle.md) |
| DI Scope per request | [02-request-lifecycle.md](./02-request-lifecycle.md) |
| Use / Run / Map / MapWhen | [03-middleware.md](./03-middleware.md) |
| Порядок middleware | [03-middleware.md](./03-middleware.md) |
| Своё middleware (IMiddleware) | [03-middleware.md](./03-middleware.md) |
| Endpoint Routing | [04-routing.md](./04-routing.md) |
| Route constraints | [04-routing.md](./04-routing.md) |
| Minimal API, Route Groups | [04-routing.md](./04-routing.md) |
| FromBody / FromQuery / FromRoute | [05-model-binding.md](./05-model-binding.md) |
| Кастомный IModelBinder | [05-model-binding.md](./05-model-binding.md) |
| FluentValidation | [05-model-binding.md](./05-model-binding.md) |
| IExceptionHandler (.NET 8+) | [06-exception-handling.md](./06-exception-handling.md) |
| Problem Details RFC 7807 | [06-exception-handling.md](./06-exception-handling.md) |
| Singleton / Scoped / Transient | [07-dependency-injection.md](./07-dependency-injection.md) |
| Captive Dependency | [07-dependency-injection.md](./07-dependency-injection.md) |
| Keyed Services (.NET 8+) | [07-dependency-injection.md](./07-dependency-injection.md) |
| IOptions vs IOptionsMonitor | [07-dependency-injection.md](./07-dependency-injection.md) |
| appsettings.json приоритеты | [08-configuration.md](./08-configuration.md) |
| User Secrets | [08-configuration.md](./08-configuration.md) |
| Environment Variables | [08-configuration.md](./08-configuration.md) |
| BackgroundService | [09-hosted-services.md](./09-hosted-services.md) |
| Graceful Shutdown | [09-hosted-services.md](./09-hosted-services.md) |
| PeriodicTimer | [09-hosted-services.md](./09-hosted-services.md) |
| IActionFilter, IAsyncActionFilter | [10-filters.md](./10-filters.md) |
| IResourceFilter (кеш на фильтре) | [10-filters.md](./10-filters.md) |
| IExceptionFilter | [10-filters.md](./10-filters.md) |
| ServiceFilter vs TypeFilter | [10-filters.md](./10-filters.md) |
| Полная диаграмма pipeline | [10-filters.md](./10-filters.md) |

## Источники

- `ИИ конспекты/aspnet-pipeline.md` — исходный конспект (~1500 строк)

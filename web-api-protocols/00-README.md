# Web API Protocols

REST, gRPC, GraphQL, OData — четыре подхода к построению API. Каждый решает разные проблемы и имеет свою цену. Здесь — принципы, устройство под капотом и .NET реализация каждого.

## Порядок чтения

1. [01-rest-design.md](./01-rest-design.md) — REST ограничения Филдинга, URL дизайн, HTTP методы, idempotency, HATEOAS
2. [02-rest-patterns.md](./02-rest-patterns.md) — Версионирование, пагинация (offset/cursor), Problem Details, ETag, Idempotency-Key
3. [03-grpc-protobuf.md](./03-grpc-protobuf.md) — Protocol Buffers: бинарное кодирование, .proto синтаксис, backward compatibility
4. [04-grpc-dotnet.md](./04-grpc-dotnet.md) — 4 типа стриминга, Grpc.AspNetCore, Interceptors, gRPC-Web
5. [05-graphql-schema.md](./05-graphql-schema.md) — Schema SDL, типы, Query/Mutation/Subscription, execution pipeline
6. [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) — N+1 + DataLoader, HotChocolate, projections, Relay пагинация
7. [07-odata.md](./07-odata.md) — Query options ($filter/$expand/$select), .NET OData, EDM, Actions/Functions
8. [08-comparison.md](./08-comparison.md) — Сравнительная таблица, дерево выбора, когда что использовать

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Принципы REST (Филдинг) | [01-rest-design.md](./01-rest-design.md) |
| URL дизайн, ресурсо-ориентированный | [01-rest-design.md](./01-rest-design.md) |
| Idempotency, PUT vs PATCH | [01-rest-design.md](./01-rest-design.md) |
| HATEOAS, Richardson maturity model | [01-rest-design.md](./01-rest-design.md) |
| Версионирование API (.NET) | [02-rest-patterns.md](./02-rest-patterns.md) |
| HTTP статус-коды | [02-rest-patterns.md](./02-rest-patterns.md) |
| Cursor-based пагинация | [02-rest-patterns.md](./02-rest-patterns.md) |
| Problem Details RFC 7807 | [02-rest-patterns.md](./02-rest-patterns.md) |
| ETag, If-None-Match, 304 | [02-rest-patterns.md](./02-rest-patterns.md) |
| Idempotency-Key middleware | [02-rest-patterns.md](./02-rest-patterns.md) |
| Protobuf бинарное кодирование | [03-grpc-protobuf.md](./03-grpc-protobuf.md) |
| .proto синтаксис, field numbering | [03-grpc-protobuf.md](./03-grpc-protobuf.md) |
| Backward compatibility в gRPC | [03-grpc-protobuf.md](./03-grpc-protobuf.md) |
| Well-known types (Timestamp, Any) | [03-grpc-protobuf.md](./03-grpc-protobuf.md) |
| Server/Client/Bidi streaming | [04-grpc-dotnet.md](./04-grpc-dotnet.md) |
| Grpc.AspNetCore сервер и клиент | [04-grpc-dotnet.md](./04-grpc-dotnet.md) |
| gRPC Interceptors (logging, auth) | [04-grpc-dotnet.md](./04-grpc-dotnet.md) |
| gRPC Health Check, Reflection | [04-grpc-dotnet.md](./04-grpc-dotnet.md) |
| gRPC-Web для браузеров | [04-grpc-dotnet.md](./04-grpc-dotnet.md) |
| GraphQL Schema SDL | [05-graphql-schema.md](./05-graphql-schema.md) |
| Interface, Union, Enum типы | [05-graphql-schema.md](./05-graphql-schema.md) |
| GraphQL execution pipeline | [05-graphql-schema.md](./05-graphql-schema.md) |
| Resolvers, partial response | [05-graphql-schema.md](./05-graphql-schema.md) |
| N+1 проблема в GraphQL | [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) |
| DataLoader (batch + dedup) | [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) |
| HotChocolate setup | [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) |
| [UseProjection], [UseFiltering] | [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) |
| Relay Cursor пагинация | [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) |
| GraphQL Subscriptions | [06-graphql-hotchocolate.md](./06-graphql-hotchocolate.md) |
| OData $filter, $expand, $select | [07-odata.md](./07-odata.md) |
| [EnableQuery] и трансляция в SQL | [07-odata.md](./07-odata.md) |
| EDM модель, $metadata | [07-odata.md](./07-odata.md) |
| OData Actions и Functions | [07-odata.md](./07-odata.md) |
| REST vs gRPC vs GraphQL vs OData | [08-comparison.md](./08-comparison.md) |
| Когда какой протокол выбрать | [08-comparison.md](./08-comparison.md) |
| Комбинирование протоколов | [08-comparison.md](./08-comparison.md) |

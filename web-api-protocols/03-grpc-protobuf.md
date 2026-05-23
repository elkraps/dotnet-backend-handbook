# gRPC: Protocol Buffers

> Protobuf — причина, по которой gRPC быстрее REST. Бинарный формат, схема как единственный источник правды, обратная совместимость через номера полей.

## Содержание
- [Почему protobuf быстрее JSON](#почему-protobuf-быстрее-json)
- [Синтаксис .proto файла](#синтаксис-proto-файла)
- [Бинарное кодирование](#бинарное-кодирование)
- [Типы полей и их кодирование](#типы-полей-и-их-кодирование)
- [Правила нумерации и обратная совместимость](#правила-нумерации-и-обратная-совместимость)
- [Well-known Types](#well-known-types)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

---

## Почему protobuf быстрее JSON

```
JSON:   { "id": 42, "name": "Alice", "active": true }
        → ~43 байта текст, требует парсинг строки

Protobuf: field_1=varint(42), field_2=string("Alice"), field_3=bool(true)
        → ~11 байт бинарь, прямое чтение памяти
```

**Три причины компактности:**
1. **Поля — числа, не строки.** `"customerId"` (10 байт ключ) → `field 3` (1 байт тег).
2. **Varint-кодирование.** Малые числа (0-127) кодируются в 1 байт, независимо от типа (`int32`, `int64`).
3. **Нет разделителей.** Длина данных закодирована в заголовке каждого поля, не через `{`, `}`, `"`.

**Производительность на практике:**
- Сериализация: ~5-10x быстрее JSON
- Размер сообщения: ~3-10x меньше JSON
- Десериализация: ~5x быстрее (нет парсинга строк)

---

## Синтаксис .proto файла

```protobuf
syntax = "proto3";               // обязательно первой строкой
option csharp_namespace = "OrderService.Grpc";  // namespace в генерированном коде
package orders;                  // пространство имён для избежания конфликтов

import "google/protobuf/timestamp.proto";  // well-known types

// Определение сервиса
service OrderService {
    rpc GetOrder    (GetOrderRequest)   returns (OrderResponse);           // unary
    rpc ListOrders  (ListOrdersRequest) returns (stream OrderResponse);    // server streaming
    rpc BatchCreate (stream CreateOrderRequest) returns (BatchResult);     // client streaming
    rpc WatchOrders (stream WatchRequest) returns (stream OrderResponse);  // bidi streaming
}

// Сообщения
message GetOrderRequest {
    int64 id = 1;
}

message CreateOrderRequest {
    int64 customer_id = 1;
    repeated OrderItem items = 2;  // repeated = массив
    optional string notes = 3;     // proto3: явная опциональность (hasField)
}

message OrderItem {
    int64 product_id = 1;
    int32 quantity   = 2;
    double price     = 3;
}

message OrderResponse {
    int64                    id          = 1;
    int64                    customer_id = 2;
    OrderStatus              status      = 3;
    repeated OrderItem       items       = 4;
    google.protobuf.Timestamp created_at = 5;  // well-known type
    map<string, string>      metadata   = 6;  // map type
}

message BatchResult {
    int32 created = 1;
    int32 failed  = 2;
}

message ListOrdersRequest {
    int64  customer_id = 1;
    int32  page_size   = 2;
    string page_token  = 3;
}

message WatchRequest {
    repeated int64 order_ids = 1;
}

// Enum — первое значение всегда 0 (UNSPECIFIED)
enum OrderStatus {
    ORDER_STATUS_UNSPECIFIED = 0;  // дефолт при десериализации неизвестного значения
    ORDER_STATUS_PENDING     = 1;
    ORDER_STATUS_CONFIRMED   = 2;
    ORDER_STATUS_SHIPPED     = 3;
    ORDER_STATUS_CANCELLED   = 4;
}
```

**Генерация C# кода из .proto:**

```xml
<!-- OrderService.csproj -->
<ItemGroup>
    <PackageReference Include="Grpc.AspNetCore" Version="2.62.0" />
    <PackageReference Include="Google.Protobuf" Version="3.27.0" />
    <PackageReference Include="Grpc.Tools" Version="2.62.0" PrivateAssets="All" />
</ItemGroup>

<ItemGroup>
    <!-- GrpcServices=Server — генерирует серверный базовый класс -->
    <Protobuf Include="Protos\orders.proto" GrpcServices="Server" />
    <!-- GrpcServices=Client — генерирует клиентский stub -->
    <!-- GrpcServices=Both   — для библиотек, используемых как сервером так и клиентом -->
</ItemGroup>
```

`Grpc.Tools` запускает `protoc` при сборке и генерирует C# файлы в `obj/` директории.

---

## Бинарное кодирование

Каждое поле в protobuf кодируется как **тег + значение**:

```
[field_tag][value_bytes]
    │
    └── field_number << 3 | wire_type
        wire_type:
          0 = Varint (int32, int64, bool, enum)
          1 = 64-bit (fixed64, double)
          2 = Length-delimited (string, bytes, message, repeated)
          5 = 32-bit (fixed32, float)
```

**Пример: `{ id: 42, name: "Alice" }`**

```
Поле id (field_number=1, wire_type=0):
  tag = (1 << 3) | 0 = 0x08
  value = varint(42) = 0x2A
  → bytes: 08 2A

Поле name (field_number=2, wire_type=2):
  tag = (2 << 3) | 2 = 0x12
  length = varint(5) = 0x05
  value = "Alice" = 41 6C 69 63 65
  → bytes: 12 05 41 6C 69 63 65

Итого: 08 2A 12 05 41 6C 69 63 65 = 9 байт
JSON:  {"id":42,"name":"Alice"} = 24 байта
```

**Varint кодирование:**

```
Число 1    → 0x01          (1 байт)
Число 127  → 0x7F          (1 байт)
Число 128  → 0x80 0x01     (2 байта) — MSB=1 означает "ещё байты следуют"
Число 300  → 0xAC 0x02     (2 байта)
INT32_MAX  → 5 байт максимум
```

Малые числа (0–127) занимают 1 байт — типично для ID, счётчиков, статусов.

---

## Типы полей и их кодирование

| Тип в .proto | Wire type | Размер | Когда использовать |
|--------------|-----------|--------|-------------------|
| `int32` / `int64` | Varint | 1-10 байт | Положительные числа |
| `sint32` / `sint64` | Varint (ZigZag) | 1-10 байт | Отрицательные числа — `sint` эффективнее `int` для отрицательных |
| `uint32` / `uint64` | Varint | 1-10 байт | Беззнаковые числа |
| `bool` | Varint | 1 байт | Булевы значения |
| `enum` | Varint | 1-10 байт | Перечисления |
| `fixed32` | 32-bit | 4 байта | Числа > 2²⁸ — эффективнее varint |
| `fixed64` | 64-bit | 8 байт | Числа > 2⁵⁶ — эффективнее varint |
| `float` | 32-bit | 4 байта | IEEE 754 float |
| `double` | 64-bit | 8 байт | IEEE 754 double |
| `string` | Length-delimited | len+1 байт | UTF-8 строка |
| `bytes` | Length-delimited | len+1 байт | Произвольные байты |
| `message` | Length-delimited | вложенное | Вложенное сообщение |

**Важно:** `int32` с отрицательным значением кодируется в **10 байт** (varint от 64-bit отрицательного числа). Используй `sint32` для полей, где возможны отрицательные значения.

---

## Правила нумерации и обратная совместимость

**Правило нумерации полей:**
- Номера 1–15: кодируются в **1 байт** → используй для часто встречающихся полей
- Номера 16–2047: кодируются в **2 байта**
- Максимум: 536 870 911 (2²⁹−1, числа 19000–19999 зарезервированы для protobuf)

**Никогда не переиспользуй удалённые номера — пометь как reserved:**

```protobuf
message Order {
    reserved 3, 5 to 8;        // удалённые номера полей
    reserved "old_status", "deprecated_field";  // удалённые имена

    int64  id     = 1;
    string name   = 2;
    // 3, 5-8 — зарезервированы, нельзя использовать снова
    double amount = 4;
}
```

**Правила backward-compatible изменений:**

```protobuf
// ✅ МОЖНО — добавить новое поле (старые клиенты игнорируют неизвестные поля)
message OrderResponse {
    int64 id     = 1;
    string name  = 2;
    string email = 3;  // новое поле — безопасно
}

// ✅ МОЖНО — переименовать поле (wire format использует номер, не имя)
// Старый: string user_name = 2;
// Новый:  string customer_name = 2;  // то же поле, другое имя — совместимо

// ❌ НЕЛЬЗЯ — изменить номер поля (старые сообщения не совместимы)
// Было:   string name = 2;
// Стало:  string name = 5;  // BREAKING CHANGE

// ❌ НЕЛЬЗЯ — изменить тип поля на несовместимый
// Было:   int32 quantity = 3;
// Стало:  string quantity = 3;  // BREAKING CHANGE
```

**Версионирование в gRPC:** используй package namespace:

```protobuf
// orders/v1/orders.proto
package orders.v1;

// orders/v2/orders.proto
package orders.v2;
// Можно запускать v1 и v2 сервисы параллельно
```

---

## Well-known Types

Google предоставляет стандартные типы для часто встречающихся данных:

```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/wrappers.proto";
import "google/protobuf/any.proto";
import "google/protobuf/struct.proto";

message Event {
    google.protobuf.Timestamp occurred_at = 1;  // UTC timestamp
    google.protobuf.Duration  duration    = 2;  // timespan

    // Nullable типы (обёртки над примитивами)
    google.protobuf.Int32Value optional_count = 3;  // null если не задан
    google.protobuf.StringValue optional_note = 4;

    // Произвольный тип с type URL
    google.protobuf.Any payload = 5;

    // Динамическая структура (как JSON object)
    google.protobuf.Struct metadata = 6;
}
```

```csharp
// Timestamp ↔ DateTime
var ts = Timestamp.FromDateTime(DateTime.UtcNow);
DateTime dt = ts.ToDateTime();

// Nullable int через wrapper
var msg = new Event { OptionalCount = 42 };          // задан
var empty = new Event { OptionalCount = null };       // не задан (HasValue = false)

// Any — упаковка произвольного сообщения
var any = Any.Pack(new OrderResponse { Id = 42 });
if (any.Is(OrderResponse.Descriptor))
{
    var order = any.Unpack<OrderResponse>();
}
```

---

## Подводные камни

**proto3: все поля optional по умолчанию, нет required.** Отсутствующее поле = default value (0, "", false, empty list). Нельзя отличить "поле не задано" от "поле = 0" — для этого используй `optional` keyword (proto3 optional) или wrapper types.

```protobuf
// proto3: как отличить 0 от "не задан"?
int32 quantity = 1;  // 0 = default, нет способа узнать, задано ли

// Решение 1: optional keyword (proto3 >= 3.15)
optional int32 quantity = 1;  // генерирует hasQuantity() метод

// Решение 2: wrapper type
google.protobuf.Int32Value quantity = 1;  // null если не задан
```

**Изменение типа `repeated` → `map` — breaking change.** Wire format несовместим.

**Enum со значением 0 должен быть "UNSPECIFIED".** При десериализации неизвестного enum значения (добавленного в новой версии) старый клиент получит 0. Если 0 = реальное значение (например, `PENDING`) — старый клиент неправильно интерпретирует.

**Большие файлы .proto в монорепо.** Один `.proto` на весь сервис → любое изменение регенерирует весь код. Разбивай по доменам: `orders.proto`, `customers.proto`, `common.proto`.

---

## См. также

- [04-grpc-dotnet.md](./04-grpc-dotnet.md) — реализация gRPC сервера и клиента в .NET
- [08-comparison.md](./08-comparison.md) — когда protobuf/gRPC vs REST/JSON

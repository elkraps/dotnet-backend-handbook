# Антипаттерны ООП и проектирования

> Сквозные проблемы, которые не привязаны к одному принципу или паттерну, а вырастают из их неправильного или чрезмерного применения. Часто встречаются на собеседованиях как «а что не так с этим кодом».

## Содержание
- [God Object](#god-object)
- [Anemic Domain Model](#anemic-domain-model)
- [Злоупотребление наследованием](#злоупотребление-наследованием)
- [Interface Bloat](#interface-bloat)
- [Pattern Overuse](#pattern-overuse)
- [Service Locator вместо DI](#service-locator-вместо-di)
- [Шпаргалка: принцип → нарушение → симптом](#шпаргалка-принцип--нарушение--симптом)
- [См. также](#см-также)

## God Object

Класс, который знает и делает слишком много: смешивает несколько несвязанных ответственностей, зависит от десятков других типов, растёт бесконтрольно.

```csharp
// God Object
public class UserManager
{
    public void Register(User user) { }
    public void SendEmail(string to, string body) { }
    public void GenerateReport(DateTime from) { }
    public void BackupDatabase() { }
    public void ValidatePassword(string password) { }
    public void ExportToCsv(IEnumerable<User> users) { }
}
```

Прямое нарушение SRP (одна причина для изменения) и GRASP High Cohesion. Типичный путь появления — «удобно добавить сюда ещё один метод», без пересмотра границ ответственности. Лечится декомпозицией по акторам: `UserService`, `EmailService`, `ReportService`, `BackupService`.

## Anemic Domain Model

Доменные объекты содержат только данные (геттеры/сеттеры), вся логика вынесена в «сервисы». Формально классы существуют, но это не ООП, а процедурный код в объектной обёртке.

```csharp
// Anemic model — Order это просто DTO
public class Order
{
    public List<LineItem> Items { get; set; } = new();
    public decimal Total { get; set; }
    public OrderStatus Status { get; set; }
}

public class OrderService
{
    public void AddItem(Order order, LineItem item)
    {
        order.Items.Add(item);
        order.Total = order.Items.Sum(i => i.Price * i.Quantity); // бизнес-правило снаружи
    }

    public void Cancel(Order order)
    {
        if (order.Status == OrderStatus.Shipped)
            throw new InvalidOperationException("Cannot cancel shipped order"); // инвариант снаружи
        order.Status = OrderStatus.Cancelled;
    }
}
```

Проблема: ничто не мешает вызывающему коду сделать `order.Status = OrderStatus.Cancelled` напрямую, в обход проверки в `OrderService.Cancel`. Инкапсуляция (см. [ООП — 4 принципа](./01-oop-principles.md)) нарушена — данные публичны и мутабельны без контроля инварианта.

Богатая модель переносит поведение внутрь:

```csharp
public class Order
{
    private readonly List<LineItem> _items = new();
    public IReadOnlyList<LineItem> Items => _items;
    public decimal Total => _items.Sum(i => i.Price * i.Quantity); // Information Expert
    public OrderStatus Status { get; private set; }

    public void AddItem(LineItem item) => _items.Add(item);

    public void Cancel()
    {
        if (Status == OrderStatus.Shipped)
            throw new InvalidOperationException("Cannot cancel shipped order");
        Status = OrderStatus.Cancelled;
    }
}
```

Anemic Domain Model — не всегда ошибка (в CRUD-приложениях с тонкой бизнес-логикой это осознанный компромисс), но в системах со сложными бизнес-правилами она рассеивает инварианты по десяткам сервисов, где их легко забыть проверить в одном из мест.

## Злоупотребление наследованием

Наследование выбирают ради переиспользования кода, а не ради настоящего «is-a» отношения. Результат — Fragile Base Class Problem (см. [ООП — 4 принципа](./01-oop-principles.md#наследование)): изменение в базовом классе непредсказуемо ломает наследников.

Признаки: глубокая иерархия (4+ уровня), наследники переопределяют методы только чтобы бросить `NotSupportedException`, `sealed` нигде не используется «на всякий случай — вдруг понадобится наследовать».

Правило: если наследник не может честно ответить на вопрос «я — это база, только более специфичный», нужна композиция, а не наследование. Классический пример — Square/Rectangle (см. [SOLID: LSP](./02-solid.md#liskov-substitution-principle)).

## Interface Bloat

Толстые интерфейсы, где реализация вынуждена бросать исключения на неподдерживаемых методах — прямое нарушение ISP (см. [SOLID](./02-solid.md#interface-segregation-principle)).

```csharp
public class SimplePrinter : IMultiFunction
{
    public void Print(Document doc) { /* OK */ }
    public void Scan(Document doc) => throw new NotSupportedException();
    public void Fax(Document doc) => throw new NotSupportedException();
}
```

Если реализация класса на треть состоит из `throw new NotSupportedException()` — интерфейс спроектирован неправильно, а не класс.

## Pattern Overuse

Применение паттерна там, где нет соответствующей проблемы — вариативности, которую нужно скрыть, или сложности, которую нужно разбить. Симптомы:

- `IFoo`/`Foo` пара, где `Foo` — единственная и единственно возможная реализация: абстракция без вариативности не даёт выгоды DIP, только косвенность.
- MediatR/CQRS на простом CRUD без cross-cutting concerns — лишний слой между контроллером и репозиторием.
- Factory Method с `switch` на 2 варианта, которые никогда не станут третьим — обычный `if` был бы читаемее.
- Decorator-цепочка из 5+ звеньев — отладка превращается в хождение по стеку маленьких классов без единой точки для брейкпоинта.

Паттерн — решение конкретной проблемы вариативности/сложности, а не чек-лист «хороший код обязан содержать паттерны».

## Service Locator вместо DI

Класс сам достаёт зависимости из глобального контейнера вместо получения их через конструктор — зависимости скрыты, тестирование требует поднятия всего контейнера.

```csharp
// Service Locator — антипаттерн
public class OrderService
{
    public async Task Place(Order order)
    {
        var repository = ServiceLocator.Get<IOrderRepository>(); // скрытая зависимость
        await repository.Save(order);
    }
}

// DI — зависимость видна в сигнатуре конструктора
public class OrderService
{
    private readonly IOrderRepository _repository;
    public OrderService(IOrderRepository repository) => _repository = repository;

    public async Task Place(Order order) => await _repository.Save(order);
}
```

Nарушает DIP (см. [SOLID](./02-solid.md#dependency-inversion-principle)) в его практическом смысле: класс явно знает про контейнер (низкоуровневую деталь), а не только про абстракцию, которая ему нужна.

## Шпаргалка: принцип → нарушение → симптом

| Принцип/паттерн | Типичное нарушение | Как заметить в коде |
|---|---|---|
| Инкапсуляция | Anemic Domain Model | Публичные сеттеры без валидации, бизнес-правила в сервисах |
| Наследование | Fragile Base Class | Наследник ломается от «невинного» рефакторинга базового класса |
| SRP / High Cohesion | God Object | Класс с методами из разных предметных областей |
| OCP | Switch по типу вместо полиморфизма | Добавление кейса = правка существующего метода |
| LSP | Square/Rectangle-подобные иерархии | Подстановка наследника ломает вызывающий код |
| ISP | Interface Bloat | `throw new NotSupportedException()` в реализации интерфейса |
| DIP | Service Locator | Класс явно обращается к контейнеру вместо конструктора |
| GoF-паттерны | Pattern Overuse | Абстракция без вариативности реализаций |

## См. также

- [ООП — 4 принципа](./01-oop-principles.md)
- [SOLID](./02-solid.md)
- [GRASP](./03-grasp.md)

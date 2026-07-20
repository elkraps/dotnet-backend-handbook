# ООП, SOLID, GRASP и паттерны проектирования

Как классы и объекты в C# устроены, зависят друг от друга и компонуются в решения типовых задач. От базовых принципов ООП до GoF-паттернов и антипаттернов, которые вырастают из их неправильного применения.

## Порядок чтения

1. [01-oop-principles.md](./01-oop-principles.md) — инкапсуляция, наследование, полиморфизм, абстракция; vtable под капотом
2. [02-solid.md](./02-solid.md) — SRP, OCP, LSP, ISP, DIP с примерами нарушения и исправления
3. [03-grasp.md](./03-grasp.md) — Information Expert, Creator, Controller, Low/High Cohesion, Polymorphism, Pure Fabrication, Indirection, Protected Variations
4. [04-gof-creational.md](./04-gof-creational.md) — Singleton, Factory Method/Abstract Factory, Builder, Prototype
5. [05-gof-structural.md](./05-gof-structural.md) — Adapter, Decorator, Facade, Proxy
6. [06-gof-behavioral.md](./06-gof-behavioral.md) — Strategy, Observer, Command, Template Method, Mediator, Chain of Responsibility
7. [07-collections.md](./07-collections.md) — concurrent-коллекции, иерархия интерфейсов, трансформации
8. [08-antipatterns.md](./08-antipatterns.md) — God Object, Anemic Domain Model, злоупотребление наследованием и паттернами

## Быстрая навигация

| Концепция | Файл |
|-----------|------|
| Инкапсуляция, information hiding | [01-oop-principles.md](./01-oop-principles.md) |
| virtual/override/new, sealed, Fragile Base Class | [01-oop-principles.md](./01-oop-principles.md) |
| Vtable, guarded devirtualization | [01-oop-principles.md](./01-oop-principles.md) |
| Abstract class vs Interface, default interface methods | [01-oop-principles.md](./01-oop-principles.md) |
| SRP, OCP | [02-solid.md](./02-solid.md) |
| LSP, ковариантность/контравариантность | [02-solid.md](./02-solid.md) |
| ISP, DIP, DI-контейнер | [02-solid.md](./02-solid.md) |
| Information Expert, Creator, Controller | [03-grasp.md](./03-grasp.md) |
| Low Coupling, High Cohesion, Pure Fabrication | [03-grasp.md](./03-grasp.md) |
| Singleton (Lazy\<T\>) | [04-gof-creational.md](./04-gof-creational.md) |
| Factory Method, Abstract Factory | [04-gof-creational.md](./04-gof-creational.md) |
| Builder, Prototype | [04-gof-creational.md](./04-gof-creational.md) |
| Adapter, Decorator (Stream, DelegatingHandler) | [05-gof-structural.md](./05-gof-structural.md) |
| Facade, Proxy (Caching/Virtual/Protection/Remote) | [05-gof-structural.md](./05-gof-structural.md) |
| Strategy, Observer (event) | [06-gof-behavioral.md](./06-gof-behavioral.md) |
| Command, Template Method | [06-gof-behavioral.md](./06-gof-behavioral.md) |
| Mediator (MediatR), Chain of Responsibility (middleware) | [06-gof-behavioral.md](./06-gof-behavioral.md) |
| ConcurrentDictionary/Queue/Bag | [07-collections.md](./07-collections.md) |
| IEnumerable/ICollection/IList иерархия | [07-collections.md](./07-collections.md) |
| ToArray/ToList/AsReadOnly/ToFrozenDictionary | [07-collections.md](./07-collections.md) |
| God Object, Anemic Domain Model | [08-antipatterns.md](./08-antipatterns.md) |
| Interface Bloat, Pattern Overuse, Service Locator | [08-antipatterns.md](./08-antipatterns.md) |

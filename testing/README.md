# Тестирование в .NET

От выбора уровня проверки до unit- и integration-тестов, изоляции данных и устройства xUnit. Примеры используют xUnit v2; доменные классы обозначают код приложения.

## Порядок чтения

| Файл | Что внутри |
|------|------------|
| [Пирамида и виды тестов](./01-test-pyramid-and-types.md) | Как выбрать уровень проверки по риску и стоимости. |
| [AAA и FIRST](./02-aaa-first.md) | Как писать небольшие воспроизводимые тесты. |
| [Test Doubles](./03-test-doubles.md) | Как изолировать зависимости и выбрать stub, fake, spy или mock. |
| [Unit-тесты в .NET](./04-unit-tests.md) | Именование, параметризация и проверка одного поведения. |
| [Данные для тестов](./05-test-data.md) | Builder, Object Mother и генераторы без случайных падений. |
| [Интеграционные тесты](./06-integration-tests.md) | Реальные БД и Redis, ASP.NET Core pipeline и HTTP-зависимости. |
| [Coverage и mutation testing](./07-coverage-and-mutation.md) | Как найти пропуски в проверках и оценить качество assertions. |
| [Устройство xUnit](./08-xunit-internals.md) | Discovery, fixtures, изоляция, parallel execution и cleanup. |
| [Антипаттерны тестирования](./09-antipatterns.md) | Почему тесты флакуют и ломаются после безопасного рефакторинга. |

## Связанные разделы

- [Кеширование](../caching/README.md)
- [ASP.NET Core Pipeline](../aspnet-pipeline/README.md)

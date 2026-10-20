# Unit-тесты в .NET

> Именование, параметризация и проверка одного поведения.

## Содержание

- [Выбор библиотек](#выбор-библиотек)
- [Именование](#именование)
- [Один assert — один тест](#один-assert--один-тест)
- [Theory и InlineData](#theory-и-inlinedata)
- [Assertions](#assertions)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## Выбор библиотек

xUnit, NUnit и MSTest подходят для .NET-тестов. Выбирай совместимый со стеком проекта фреймворк; особенности изоляции, parallel execution и жизненного цикла проверь для используемой версии.

Здесь примеры рассчитаны на xUnit v2: `[Fact]`, `[Theory]`, `Assert` и `IAsyncLifetime` с `Task`. Доменные классы (`Order`, `Product`, `AppDbContext`) обозначают код приложения; фрагменты не являются отдельным готовым проектом.

Для нового тестового проекта нужны runner/SDK, совместимые с выбранной версией xUnit. FluentAssertions — дополнительная библиотека: проверь лицензию версии перед добавлением. В большинстве примеров хватает стандартного `Assert`.

[Документация xUnit](https://xunit.net/docs/shared-context).

## Именование

Формат: `MethodName_Scenario_ExpectedResult`

```csharp
// Хорошо
Calculate_WithTenPercentDiscount_ReducesTotalByTen()
Process_WithExpiredCard_ThrowsPaymentException()
Find_WithNonExistentId_ReturnsNull()
Validate_WithEmptyEmail_ReturnsFalse()

// Плохо
Test1()
CalculateTest()
TestCalculateDiscount()
ShouldCalculate()
```

## Один assert — один тест

```csharp
// Плохо — несколько проверок, при падении непонятно что сломалось
[Fact]
public void Process_ValidOrder_ProcessesCorrectly()
{
    var result = service.Process(order);
    Assert.Equal(OrderStatus.Completed, result.Status);
    Assert.NotNull(result.InvoiceId);
    Assert.True(result.ProcessedAt > DateTime.UtcNow.AddSeconds(-1));
}

// Хорошо — каждый аспект отдельно
[Fact]
public void Process_ValidOrder_SetsStatusCompleted()
{
    var result = _service.Process(CreateValidOrder());
    Assert.Equal(OrderStatus.Completed, result.Status);
}

[Fact]
public void Process_ValidOrder_AssignsInvoiceId()
{
    var result = _service.Process(CreateValidOrder());
    Assert.NotNull(result.InvoiceId);
}
```

## Theory и InlineData

```csharp
[Theory]
[InlineData(100, 10, 90)]
[InlineData(200, 25, 150)]
[InlineData(50,  0,  50)]
[InlineData(0,   50, 0)]
public void Calculate_WithDiscount_ReturnsCorrectTotal(
    decimal price, decimal discountPercent, decimal expected)
{
    var service = new PricingService();
    var order = new Order { BasePrice = price, DiscountPercent = discountPercent };

    var result = service.Calculate(order);

    Assert.Equal(expected, result.Total);
}
```

## Assertions

**FluentAssertions** — читаемые ошибки и цепочки проверок:

```csharp
// MSTest / xUnit стандарт
Assert.Equal(expected, actual);
Assert.True(collection.Contains(item));

// FluentAssertions
actual.Should().Be(expected, because: "discount must reduce total");
collection.Should().Contain(item).And.HaveCount(3);
result.Should().BeOfType<OrderCreatedEvent>()
      .Which.OrderId.Should().Be(expectedId);
```

Сообщение FluentAssertions показывает ожидаемое и фактическое значения с контекстом, что облегчает разбор падения.

## Подводные камни

Один assert — полезный ориентир. Несколько независимых свойств лучше проверять отдельными тестами; образцы плохих тестов намеренно нарушают это правило.

## См. также

- [AAA и FIRST](./02-aaa-first.md)
- [Данные для тестов](./05-test-data.md)
- [Оглавление раздела](./README.md)

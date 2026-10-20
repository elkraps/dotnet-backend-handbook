# Инвалидация и консистентность

> Почему обновление БД и удаление кеша не исключают stale data.

## Содержание

- [Stale data и инвалидация](#stale-data-и-инвалидация)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## Stale data и инвалидация

**Инвалидация по событию** (лучший подход):

```csharp
public async Task UpdateProductAsync(Product product, CancellationToken ct)
{
    _db.Update(product);
    await _db.SaveChangesAsync(ct);

    // Инвалидируем кеш сразу после изменения
    await _cache.RemoveAsync($"product:{product.Id}", ct);

    // Можно сразу прогреть
    await _cache.SetStringAsync($"product:{product.Id}",
        JsonSerializer.Serialize(product),
        new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30) },
        ct);
}
```

**Инвалидация через Pub/Sub** (для нескольких инстансов):

```csharp
// Publisher (при изменении данных)
await _redis.GetSubscriber().PublishAsync("cache-invalidation", $"product:{id}");

// Subscriber (в каждом инстансе приложения)
await _redis.GetSubscriber().SubscribeAsync("cache-invalidation", (channel, message) =>
{
    _memoryCache.Remove(message.ToString());
});
```

### Гонка читателя и писателя

1. Читатель получает старое значение из БД, но ещё не записал его в кеш.
2. Писатель сохраняет новое значение и удаляет кеш.
3. Читатель записывает старое значение уже после удаления.

TTL ограничивает время устаревания, но не устраняет гонку. Для более строгих требований нужны версии записей, согласованный порядок операций или проверка версии перед заполнением. Сбой после commit БД и до invalidation обрабатывают повторной доставкой события, например через Outbox.

Pub/Sub не сохраняет события: отключившийся процесс может пропустить invalidation. У L1 должен быть конечный TTL и стратегия восстановления. Быстрое повторное заполнение в примере также может конкурировать с другой записью; оно не даёт строгой консистентности.


## Подводные камни

Обработчик события должен быть идемпотентным. Учитывай порядок версий и пропущенные Pub/Sub-сообщения.

## См. также

- [Стратегии кеширования](./02-cache-strategies.md)
- [Практические паттерны Redis](./09-practical-patterns.md)
- [Оглавление раздела](./README.md)

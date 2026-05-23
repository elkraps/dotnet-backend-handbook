# Fluent API vs Data Annotations: конфигурация EF Core

> В DDD доменные сущности не должны зависеть от EF Core. Fluent API через IEntityTypeConfiguration<T> держит конфигурацию отдельно от домена. Data Annotations загрязняют доменный слой инфраструктурными деталями.

## Содержание
- [Приоритет конфигурации](#приоритет-конфигурации)
- [Когда Data Annotations, когда Fluent API](#когда-data-annotations-когда-fluent-api)
- [IEntityTypeConfiguration: структура](#ientitytypeconfiguration-структура)
- [Индексы](#индексы)
- [Value Conversion](#value-conversion)
- [Global Query Filter (Soft Delete)](#global-query-filter-soft-delete)
- [Owned Types](#owned-types)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

---

## Приоритет конфигурации

```
Fluent API  >  Data Annotations  >  Conventions
(высший)       (средний)            (низший)
```

**Conventions** — соглашения по умолчанию:
- Свойство `Id` или `{TypeName}Id` → первичный ключ
- `string` → `nvarchar(max)` / `text`
- Nullable reference type `?` → nullable столбец

**Data Annotations** — атрибуты `System.ComponentModel.DataAnnotations` и `Microsoft.EntityFrameworkCore`:

```csharp
[Table("products")]
[Index(nameof(Sku), IsUnique = true)]
public class Product
{
    [Key]
    public int Id { get; set; }

    [Required]
    [MaxLength(200)]
    public string Name { get; set; } = string.Empty;

    [Column(TypeName = "decimal(18,2)")]
    public decimal Price { get; set; }
}
```

**Fluent API** — в `OnModelCreating` или через `IEntityTypeConfiguration<T>`:

```csharp
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("products");
        builder.HasKey(p => p.Id);
        builder.HasIndex(p => p.Sku).IsUnique();
        builder.Property(p => p.Name).IsRequired().HasMaxLength(200);
        builder.Property(p => p.Price).HasColumnType("decimal(18,2)");
    }
}
```

---

## Когда Data Annotations, когда Fluent API

**Data Annotations уместны для:**
- Простых ограничений, привязанных к свойству: `[Required]`, `[MaxLength]`, `[StringLength]`
- Валидации, которая нужна одновременно в EF Core и ASP.NET Model Validation
- Простых read-model классов без доменной логики

**Fluent API необходим для:**

```csharp
// 1. Составной первичный ключ
builder.HasKey(e => new { e.OrderId, e.ProductId });

// 2. Конфигурация связей
builder.HasOne(o => o.Customer).WithMany(c => c.Orders)...

// 3. Фильтрованные индексы (partial index в PostgreSQL)
builder.HasIndex(o => o.Email)
    .IsUnique()
    .HasFilter("\"IsDeleted\" = false");

// 4. Owned Types, table splitting, TPH/TPT/TPC
// 5. Global query filters
// 6. Value conversions со сложной логикой
// 7. DDD-сущности, которые не должны знать об EF
```

**DDD-аргумент:** Data Annotations загрязняют домен инфраструктурными деталями.

```csharp
// WRONG: Order знает об EF Core и DataAnnotations
[Table("orders")]
public class Order
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required]
    [ForeignKey(nameof(Customer))]
    public int CustomerId { get; set; }
}

// CORRECT: Order — чистая доменная сущность
public class Order
{
    public int Id { get; private set; }
    public int CustomerId { get; private set; }
}

// Вся конфигурация отдельно
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("orders");
        builder.HasKey(o => o.Id);
        builder.Property(o => o.CustomerId).IsRequired();
    }
}
```

---

## IEntityTypeConfiguration: структура

```csharp
/// <summary>
/// Entity type configuration for the Order entity.
/// Separates EF Core infrastructure from domain model.
/// </summary>
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        // Таблица и схема
        builder.ToTable("orders", schema: "sales");

        // Первичный ключ
        builder.HasKey(o => o.Id);

        // Свойства
        builder.Property(o => o.Number)
            .IsRequired()
            .HasMaxLength(50)
            .HasColumnName("order_number");

        builder.Property(o => o.Total)
            .HasColumnType("decimal(18,2)");

        // Связи
        builder.HasOne(o => o.Customer)
            .WithMany(c => c.Orders)
            .HasForeignKey(o => o.CustomerId)
            .OnDelete(DeleteBehavior.Restrict);

        // Backing field для коллекции
        builder.Navigation(o => o.Items)
            .UsePropertyAccessMode(PropertyAccessMode.Field);
    }
}

// Регистрация: применить все конфигурации из сборки
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
}
```

---

## Индексы

```csharp
// Простой индекс
builder.HasIndex(o => o.CreatedAt);

// Уникальный индекс
builder.HasIndex(o => o.Number).IsUnique();

// Составной индекс (порядок полей важен для left-prefix rule)
builder.HasIndex(o => new { o.CustomerId, o.Status });
// SQL: CREATE INDEX ON orders (customer_id, status)
// Используется для: WHERE CustomerId = X, WHERE CustomerId = X AND Status = Y
// НЕ используется для: WHERE Status = Y (нет left-prefix)

// Фильтрованный индекс (partial index в PostgreSQL)
builder.HasIndex(o => o.Email)
    .IsUnique()
    .HasFilter("\"IsDeleted\" = false");
// Уникальность только среди не удалённых записей
// Позволяет иметь несколько удалённых записей с одним email

// Covering index (INCLUDE в PostgreSQL/SQL Server)
builder.HasIndex(o => o.CustomerId)
    .IncludeProperties(o => new { o.Total, o.Status });
// Index Only Scan без обращения к heap для запросов: WHERE CustomerId = X → SELECT Total, Status

// Именованный индекс
builder.HasIndex(o => o.CreatedAt)
    .HasDatabaseName("IX_Orders_CreatedAt_Covering")
    .IncludeProperties(o => new { o.Status, o.Total });
```

---

## Value Conversion

Преобразование значений между C# типами и типами столбцов БД.

```csharp
// Enum → string (читаемые значения в БД)
builder.Property(o => o.Status)
    .HasConversion<string>()
    .HasMaxLength(50);
// БД хранит: "Pending", "Confirmed", "Cancelled"
// C# видит: OrderStatus.Pending

// Список → строка с разделителем
builder.Property(o => o.Tags)
    .HasConversion(
        v => string.Join(',', v),                                          // C# → DB
        v => v.Split(',', StringSplitOptions.RemoveEmptyEntries).ToList()) // DB → C#
    .HasMaxLength(500);

// Value Object (Money) в один столбец
public record Money(decimal Amount, string Currency);

builder.Property(o => o.Price)
    .HasConversion(
        v => v.Amount,                       // C# → DB (только Amount)
        v => new Money(v, "USD"));           // DB → C# (Currency фиксирована)

// Кастомный ValueConverter
var converter = new ValueConverter<DateOnly, DateTime>(
    v => v.ToDateTime(TimeOnly.MinValue),   // C# DateOnly → DB DateTime
    v => DateOnly.FromDateTime(v));         // DB DateTime → C# DateOnly

builder.Property(o => o.OrderDate)
    .HasConversion(converter);
```

---

## Global Query Filter (Soft Delete)

Автоматически добавляет `WHERE` к каждому запросу — для soft delete, multi-tenancy, etc.

```csharp
// Конфигурация
builder.HasQueryFilter(o => !o.IsDeleted);

// Все запросы автоматически фильтруют удалённые
var orders = await dbContext.Orders.ToListAsync();
// SQL: SELECT * FROM orders WHERE "IsDeleted" = false

var order = await dbContext.Orders.FindAsync(42);
// SQL: SELECT * FROM orders WHERE "IsDeleted" = false AND "Id" = 42
// Если order с Id=42 удалён (IsDeleted=true) → вернёт null

// Обойти фильтр явно
var allIncludingDeleted = await dbContext.Orders
    .IgnoreQueryFilters()
    .ToListAsync();
// SQL: SELECT * FROM orders (без WHERE IsDeleted)
```

**Multi-tenancy через Global Query Filter:**

```csharp
public class AppDbContext : DbContext
{
    private readonly int _tenantId;

    public AppDbContext(DbContextOptions<AppDbContext> options, ITenantProvider tenant)
        : base(options)
    {
        _tenantId = tenant.CurrentTenantId;
    }

    public DbSet<Order> Orders { get; set; } = null!;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Каждый запрос к Orders автоматически фильтрует по TenantId
        modelBuilder.Entity<Order>()
            .HasQueryFilter(o => o.TenantId == _tenantId);
    }
}
```

---

## Owned Types

Owned type — Value Object, который хранится в столбцах той же таблицы, что и owner. Не имеет собственного DbSet и первичного ключа.

```csharp
/// <summary>
/// Value object representing a monetary amount with currency.
/// Stored as columns in the owning entity's table.
/// </summary>
public class Money
{
    public decimal Amount { get; set; }
    public string Currency { get; set; } = "USD";
}

/// <summary>
/// Address value object. Stored inline in Customer table.
/// </summary>
public class Address
{
    public string Street { get; set; } = string.Empty;
    public string City { get; set; } = string.Empty;
    public string Country { get; set; } = string.Empty;
}

// Конфигурация
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        // Money хранится в столбцах Price_Amount и Price_Currency
        builder.OwnsOne(o => o.Price, money =>
        {
            money.Property(m => m.Amount)
                 .HasColumnName("Price")
                 .HasColumnType("decimal(18,2)");
            money.Property(m => m.Currency)
                 .HasColumnName("PriceCurrency")
                 .HasMaxLength(3);
        });

        // Address хранится в столбцах ShippingStreet, ShippingCity, ShippingCountry
        builder.OwnsOne(o => o.ShippingAddress, addr =>
        {
            addr.Property(a => a.Street).HasColumnName("ShippingStreet").HasMaxLength(200);
            addr.Property(a => a.City).HasColumnName("ShippingCity").HasMaxLength(100);
            addr.Property(a => a.Country).HasColumnName("ShippingCountry").HasMaxLength(2);
        });
    }
}
// Таблица orders: Id, ..., Price, PriceCurrency, ShippingStreet, ShippingCity, ShippingCountry
```

Owned types загружаются вместе с owner автоматически — не нужен `Include`.

---

## Подводные камни

**Global Query Filter и `FindAsync`.** `FindAsync` сначала смотрит в identity map, не учитывая QueryFilter. Если сущность уже загружена без фильтра (через `IgnoreQueryFilters()`), `FindAsync` вернёт её, даже если она IsDeleted.

**Value Conversion и LINQ.** Некоторые конверсии EF не может транслировать в SQL:

```csharp
// WRONG: EF не умеет транслировать List<string> Contains через конвертер
var orders = await dbContext.Orders
    .Where(o => o.Tags.Contains("urgent"))  // Tags — List<string> с конвертером
    .ToListAsync();  // может упасть или выполниться на клиенте

// CORRECT: отдельная таблица для тегов или JSON-столбец (PostgreSQL)
builder.Property(o => o.Tags)
    .HasColumnType("jsonb");  // PostgreSQL JSON
```

**Owned Types и null.** Если owned type объявлен как nullable (`Address?`), EF Core при загрузке создаст объект с null-значениями (не вернёт null сам объект), если все столбцы NULL:

```csharp
// Явное указание nullable owned type
builder.OwnsOne(o => o.ShippingAddress, addr =>
{
    addr.Property(a => a.Street).IsRequired(false);
});
// Если все поля Address в БД NULL → o.ShippingAddress может быть null
```

**Миграции и переименование столбцов.** Изменение `HasColumnName` в Fluent API генерирует `RENAME COLUMN` в миграции. На большой таблице это может быть долгой блокирующей операцией.

---

## См. также

- [06-relationships.md](./06-relationships.md) — конфигурация связей (HasOne/HasMany)
- [02-efcore-architecture.md](./02-efcore-architecture.md) — DbContext, OnModelCreating, ApplyConfigurationsFromAssembly
- [04-projection-notracking.md](./04-projection-notracking.md) — как QueryFilter влияет на запросы

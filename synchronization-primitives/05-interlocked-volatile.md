# Interlocked, volatile и lock-free алгоритмы

> Атомарные операции — самый быстрый способ синхронизации для простых случаев. CAS-цикл — основа всех lock-free структур данных.

## Содержание
- [Interlocked](#interlocked)
- [volatile keyword](#volatile-keyword)
- [Volatile.Read / Volatile.Write](#volatileread--volatilewrite)
- [Lock-free алгоритмы: CAS loop](#lock-free-алгоритмы-cas-loop)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

---

## Interlocked

**Что это:** набор атомарных операций с полным memory barrier. Реализуются через специальные CPU-инструкции (`lock cmpxchg`, `lock xadd`), которые блокируют cache line на время операции. Стоимость ~5-10 нс.

```csharp
private long _counter;
private object _reference;

/// <summary>
/// Atomic increment. Returns the NEW value after increment.
/// </summary>
void IncrementExample()
{
    long newValue = Interlocked.Increment(ref _counter);
    // Atomic: read + add 1 + write как единая операция
    // Full memory barrier гарантирует видимость всем потокам
}

/// <summary>
/// Atomic exchange. Sets new value, returns OLD value.
/// </summary>
void ExchangeExample()
{
    // Atomically: old = _reference; _reference = newData; return old;
    object previous = Interlocked.Exchange(ref _reference, new Data());
}

/// <summary>
/// Atomic add. Returns the NEW value.
/// </summary>
void AddExample()
{
    long result = Interlocked.Add(ref _counter, 10);
}

/// <summary>
/// Atomic 64-bit read. Prevents torn reads on 32-bit platforms.
/// </summary>
void ReadExample()
{
    long value = Interlocked.Read(ref _counter);
    // На 32-bit: предотвращает torn read 64-bit значения
    // На 64-bit: эквивалентен volatile read (aligned 64-bit атомарны)
}

/// <summary>
/// Compare-and-swap (CAS) — foundation of all lock-free algorithms.
/// If _counter == comparand, set to newValue and return comparand.
/// Otherwise return current value without modification.
/// </summary>
void CompareExchangeExample()
{
    long original = Interlocked.CompareExchange(
        ref _counter,
        newValue: 42,
        comparand: 0
    );

    if (original == 0)
        Console.WriteLine("Successfully changed 0 → 42");
    else
        Console.WriteLine($"_counter was {original}, not changed");
}
```

**Все Interlocked-операции гарантируют:** атомарность + полный memory barrier (acquire + release). Это значит — никаких torn reads, никакой visibility проблемы, никакого reordering через точку вызова.

---

## volatile keyword

**Что это:** модификатор поля, добавляющий acquire-семантику к каждому чтению и release-семантику к каждой записи.

**Что гарантирует:**
- Компилятор/JIT не закеширует значение в регистре
- Чтения/записи не переупорядочиваются через volatile-доступ

**Что НЕ гарантирует:**
- Атомарность read-modify-write (`_vol++` — не атомарно)
- Атомарность для `long`/`double` на 32-bit
- Полный memory barrier (только half-fence: read = acquire, write = release)

```csharp
private volatile bool _canceled;

/// <summary>
/// Volatile is sufficient for simple flag: single writer, multiple readers.
/// </summary>
void Worker()
{
    while (!_canceled) // volatile read: всегда актуальное значение
    {
        DoWork();
    }
}

void Cancel()
{
    _canceled = true; // volatile write: немедленно видно всем потокам
}

// НЕПРАВИЛЬНО: volatile не делает ++ атомарным
private volatile int _state;
void WrongIncrement()
{
    _state++; // три операции: read → increment → write
              // Нужно: Interlocked.Increment(ref _state)
}
```

**Паттерн publish/subscribe — правильное использование volatile:**

```csharp
private int _data;
private volatile bool _ready;

// Thread 1 (publisher):
_data = 42;       // обычная запись
_ready = true;    // volatile write (release): всё выше НЕ опустится ниже этой точки
                  // Гарантия: _data = 42 видна любому, кто прочитает _ready = true

// Thread 2 (subscriber):
if (_ready)       // volatile read (acquire): всё ниже НЕ поднимется выше этой точки
{
    Use(_data);   // гарантированно 42
}
```

**Допустимые типы для `volatile`:**
- Ссылочные типы
- `bool`, `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `char`, `float`
- Enum на базе byte/short/int
- `IntPtr`, `UIntPtr`

**Нельзя:** `long`, `ulong`, `double`, `decimal` — чтение/запись не атомарны на всех платформах.

---

## Volatile.Read / Volatile.Write

Дают ту же семантику, что `volatile` keyword, но для **любого** поля (не помеченного `volatile`). Полезно когда volatile-семантика нужна только в некоторых местах, или для типов, недопустимых с `volatile`.

```csharp
private int _data;
private bool _ready; // НЕ помечено volatile

/// <summary>
/// Targeted memory ordering without marking entire field volatile.
/// </summary>
void Publish()
{
    _data = 42;
    Volatile.Write(ref _ready, true); // release barrier
                                      // _data = 42 гарантированно видна
                                      // любому, кто прочитает _ready = true
}

void Consume()
{
    if (Volatile.Read(ref _ready))    // acquire barrier
    {
        Console.WriteLine(_data);     // гарантированно 42
    }
}
```

**Когда `Volatile.Read/Write` лучше `volatile` keyword:**

1. volatile-семантика нужна только в конкретном месте, а не на каждый доступ к полю
2. Тип не поддерживает `volatile` keyword (`long`, `double`)
3. Generic-поля

**Важно:** `Volatile.Read(ref long)` **не делает чтение атомарным** на 32-bit. Только добавляет acquire-семантику. Для атомарного чтения `long` на 32-bit — `Interlocked.Read`.

---

## Lock-free алгоритмы CAS loop

**Что это:** алгоритмы без блокировок. Вместо `lock` — атомарный `Interlocked.CompareExchange` в цикле retry.

**CAS loop — базовый паттерн:**

```csharp
private long _balance;

/// <summary>
/// Lock-free withdrawal: read current, compute new, atomically swap if unchanged.
/// Retries if another thread modified value between read and swap.
/// </summary>
bool Withdraw(long amount)
{
    SpinWait spinner = new();
    while (true)
    {
        long current = Interlocked.Read(ref _balance);
        if (current < amount)
            return false;

        long desired = current - amount;
        long original = Interlocked.CompareExchange(
            ref _balance,
            desired,   // новое значение
            current    // ожидаемое текущее
        );

        if (original == current)
            return true; // success: _balance был 'current', теперь 'desired'

        // Другой поток изменил _balance между Read и CAS — повторяем
        spinner.SpinOnce();
    }
}
```

**Как работает:**
1. Прочитать текущее значение
2. Вычислить новое значение
3. CAS: «если значение всё ещё `current`, заменить на `desired`»
4. Если CAS не прошёл — повторить с актуальным значением

**Lock-free стек — классический пример:**

```csharp
/// <summary>
/// Lock-free stack using CAS on head reference.
/// </summary>
public class LockFreeStack<T>
{
    private class Node
    {
        public T Value;
        public Node Next;
    }

    private Node _head;

    public void Push(T value)
    {
        var node = new Node { Value = value };
        SpinWait spinner = new();
        while (true)
        {
            node.Next = _head;
            // Если _head не изменился с момента чтения — установить новую голову
            if (Interlocked.CompareExchange(ref _head, node, node.Next) == node.Next)
                return;
            spinner.SpinOnce();
        }
    }

    public bool TryPop(out T value)
    {
        SpinWait spinner = new();
        while (true)
        {
            Node head = _head;
            if (head == null) { value = default; return false; }

            if (Interlocked.CompareExchange(ref _head, head.Next, head) == head)
            {
                value = head.Value;
                return true;
            }
            spinner.SpinOnce();
        }
    }
}
```

**ABA-проблема:** CAS проверяет **значение**, не историю. Если значение было A → B → A, CAS не заметит изменения. Для чисел это обычно не проблема (итог тот же), но для указателей/ссылок опасно. Решение: использовать счётчик версий (stamped reference) или `Interlocked.CompareExchange` с тегом.

**Когда lock-free оправдан:**
- Очень высокий contention (миллионы операций/сек)
- Критические секции тривиальны (1-2 операции)
- Требования к latency (нет context switch)

**Когда НЕ оправдан:**
- Сложная логика (CAS retry с большими вычислениями = waste CPU)
- Низкий contention (lock дешевле и проще)
- Нужна композиция нескольких переменных (CAS работает только с одной)

---

## Подводные камни

**`Interlocked.CompareExchange` с float/double** — работает, но осторожно с NaN:

```csharp
// NaN != NaN в floating-point → CAS с NaN comparand никогда не совпадёт
float current = float.NaN;
Interlocked.CompareExchange(ref current, 1.0f, float.NaN); // никогда не сработает!
```

**`Interlocked.Increment` на `int` vs `long`** — на 32-bit обе перегрузки атомарны (разные CPU инструкции):

```csharp
int i = 0;
Interlocked.Increment(ref i);    // OK: lock xadd [mem], 1

long l = 0;
Interlocked.Increment(ref l);    // OK: lock cmpxchg8b на x86
```

**`volatile` на массивы** — делает атомарным доступ к **ссылке** на массив, но не к элементам:

```csharp
private volatile int[] _array; // атомарна смена _array на другой массив
                                // НО _array[i]++ не атомарно
```

**Не использовать `Thread.VolatileRead`/`Thread.VolatileWrite`** — устаревшие API. Используйте `Volatile.Read`/`Volatile.Write`.

---

## См. также

- [01-fundamentals.md](./01-fundamentals.md) — atomicity, visibility, ordering: три проблемы, которые решает Interlocked
- [02-lock-monitor.md](./02-lock-monitor.md) — lock как альтернатива для сложных критических секций
- [07-concurrent-collections.md](./07-concurrent-collections.md) — ConcurrentDictionary/Queue используют CAS внутри

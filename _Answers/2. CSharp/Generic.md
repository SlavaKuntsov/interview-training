Обобщённые типы позволяют создавать классы, структуры, интерфейсы и методы, параметризованные типами. Использование Generics повышает типобезопасность, производительность и переиспользование кода.

## Зачем нужны обобщения
- **Типобезопасность**: ошибки типов выявляются на этапе компиляции, а не в рантайме.
- **Производительность**: для значимых типов нет boxing/unboxing.
- **Переиспользование**: один код для разных типов данных.

## Объявление обобщённых классов, структур и интерфейсов
```csharp
public class Repository<T>
{
    private readonly List<T> _items = new List<T>();

    public void Add(T item)
    {
        _items.Add(item);
    }

    public IEnumerable<T> GetAll()
    {
        return _items;
    }
}

public struct Pair<TFirst, TSecond>
{
    public TFirst First { get; }
    public TSecond Second { get; }

    public Pair(TFirst first, TSecond second)
    {
        First = first;
        Second = second;
    }
}

public interface IService<T>
{
    void Process(T input);
}
````

## Ограничения (constraints)

Позволяют накладывать требования на параметр типа.

- `where T : struct` — value type (кроме Nullable).
    
- `where T : class` — reference type.
    
- `where T : new()` — публичный конструктор без параметров.
    
- `where T : BaseType` — наследник BaseType.
    
- `where T : interface1, interface2` — реализует интерфейсы.
    

```csharp
public class Cache<T> where T : class, new()
{
    private readonly Dictionary<string, T> _store = new();

    public T GetOrCreate(string key)
    {
        if (!_store.TryGetValue(key, out var value))
        {
            value = new T();
            _store[key] = value;
        }
        return value;
    }
}
```

## Обобщённые методы

```csharp
public static T Max<T>(T a, T b) where T : IComparable<T>
{
    return a.CompareTo(b) >= 0 ? a : b;
}

// Использование
int i = Max(10, 20);        // T выведен как int
string s = Max("a", "b");   // T выведен как string
```

## Covariance и Contravariance

Позволяют более гибко работать с совместимостью обобщённых интерфейсов и делегатов.

- **Covariant (out)**: позволяет использовать более производный тип вместо базового.
    
- **Contravariant (in)**: позволяет использовать более базовый тип вместо производного.
    

```csharp
public interface IEnumerable<out T> { /* ... */ }
public interface IComparer<in T> { int Compare(T x, T y); }

IEnumerable<string> strings = new List<string>();
IEnumerable<object> objects = strings; // Covariance

IComparer<object> compObj = GetComparer<object>();
IComparer<string> compStr = compObj;   // Contravariance
```

## Обобщённые делегаты

- `Func<T, TResult>` и `Action<T>`
    
- Пользовательские:
    

```csharp
public delegate R Transformer<T, R>(T input);
```

## Особенности CLR и Generics

- JIT генерирует специализированный код для каждого value-type, избегая boxing.
    
- Для ссылочных типов используется один общий код.
    
- Metadata содержит описания параметризованных типов.
    

## Ограничения и рекомендации

- **Нельзя** создавать массивы параметризованных типов напрямую (`new T[10]` невозможно).
    
- **Нельзя** обращаться к параметру типа как к `typeof(T)` внутри метода, если нет `where T : class`.
    
- У обобщённых классов с разными типами T — свои статические поля.
    
- Избегайте **over-constraining**, держа API проще.
    

## Generic и производительность

- Нет boxing для value-type при использовании `List<int>` vs `ArrayList`.
    
- specialization JIT для каждого T улучшает производительность.
    
- Используйте `Span<T>` и `Memory<T>` для работы с буферами эффективно.
    

## Продвинутый пример: кэш с политикой истечения

```csharp
public class ExpiringCache<TKey, TValue>
    where TKey : notnull
{
    private readonly TimeSpan _ttl;
    private readonly Dictionary<TKey, (TValue Value, DateTime Expiry)> _store = new();

    public ExpiringCache(TimeSpan ttl)
    {
        _ttl = ttl;
    }

    public TValue GetOrAdd(TKey key, Func<TKey, TValue> factory)
    {
        if (_store.TryGetValue(key, out var entry) && entry.Expiry > DateTime.UtcNow)
            return entry.Value;

        var value = factory(key);
        _store[key] = (value, DateTime.UtcNow + _ttl);
        return value;
    }
}
```

Generics — ключевой инструмент .NET, обеспечивающий надежные, быстрые и гибкие решения для любых сценариев разработки.
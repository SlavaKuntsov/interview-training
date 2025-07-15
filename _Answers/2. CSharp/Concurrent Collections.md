### Общее
Коллекции из пространства имён `System.Collections.Concurrent` спроектированы для безопасной работы нескольких потоков одновременно без внешней синхронизации (`lock`). Они используют внутренние механизмы блокировок с минимальным конфликтом и оптимизации на уровне сегментирования, позволяя достичь высокой производительности в многопоточных сценариях.

---

## ConcurrentDictionary<TKey, TValue>

Хеш-таблица, безопасная для одновременного чтения и записи.

```csharp
var dict = new ConcurrentDictionary<string, int>();

// Добавить, если ключ отсутствует
dict.TryAdd("Alice", 30);

// Обновить значение с помощью фабричной функции
dict.AddOrUpdate("Alice",
    addValue: 1,
    updateValueFactory: (key, existing) => existing + 1);

// Получить значение или задать по умолчанию
int age = dict.GetOrAdd("Bob", key => 18);

// Удалить
dict.TryRemove("Alice", out int removedValue);

// Попытаться обновить условно
dict.TryUpdate("Bob", newValue: 20, comparisonValue: 18);
````

**Особенности:**

- Использует сегментированную блокировку (lock striping) для уменьшения конкуренции.
    
- Методы `AddOrUpdate`, `GetOrAdd` выполняются атомарно.
    
- При большом числе потоков показывает значительно лучшие характеристики, чем `lock` вокруг обычного `Dictionary`.
    

---

## ConcurrentQueue и ConcurrentStack

### ConcurrentQueue (FIFO)

```csharp
var queue = new ConcurrentQueue<int>();

queue.Enqueue(1);
queue.Enqueue(2);

if (queue.TryDequeue(out int result))
{
    // result == 1
}

// Можно посмотреть без удаления
if (queue.TryPeek(out int head))
{
    // head == 2
}
```

### ConcurrentStack (LIFO)

```csharp
var stack = new ConcurrentStack<string>();

stack.Push("first");
stack.Push("second");

if (stack.TryPop(out string value))
{
    // value == "second"
}

if (stack.TryPeek(out string top))
{
    // top == "first"
}
```

**Особенности:**

- Применяют lock-free алгоритмы на основе CAS (compare-and-swap) или минимальную блокировку.
    
- Высокая пропускная способность при большом числе производителей/потребителей.
    
- Методы `TryDequeue`/`TryPop` и `TryPeek` возвращают `false`, если коллекция пуста.
    

---

## ConcurrentBag

Неупорядоченное хранилище элементов, оптимизированное для сценария “производитель–потребитель” без необходимости строгого порядка.

```csharp
var bag = new ConcurrentBag<double>();

bag.Add(3.14);
bag.Add(2.71);

if (bag.TryTake(out double item))
{
    // item может быть 2.71 или 3.14
}

// Получить все элементы (неудаляя)
double[] snapshot = bag.ToArray();
```

**Особенности:**

- Каждый поток имеет собственный локальный буфер; при пустоте локального буфера потоки «воруют» из буферов других потоков (work-stealing).
    
- Минимизирует конкуренцию при добавлении и извлечении, если потоки преимущественно работают со своими элементами.
    
- Порядок выдачи не гарантируется.
    

---

## BlockingCollection

Обёртка над любой `IProducerConsumerCollection<T>` (по умолчанию — `ConcurrentQueue<T>`), предоставляющая блокирующие методы:

```csharp
var bc = new BlockingCollection<string>(boundedCapacity: 100);

// Producer
Task.Run(() =>
{
    foreach (var line in File.ReadLines("input.txt"))
    {
        bc.Add(line); // блокируется, если достигнут предел
    }
    bc.CompleteAdding();
});

// Consumer
foreach (var item in bc.GetConsumingEnumerable())
{
    Process(item);
}
```

**Особенности:**

- `Add()` и `Take()` могут блокировать поток, если очередь полна или пуста соответственно.
    
- `GetConsumingEnumerable()` автоматически завершится, когда `CompleteAdding()` вызвано и все элементы извлечены.
    
- Поддерживает ограничение ёмкости (boundedCapacity), что защищает от бесконтрольного роста памяти.
    

---

## Подводные камни и рекомендации

- Несмотря на потокобезопасность, при комбинировании нескольких операций над коллекцией (например, проверка + обновление) всё равно может требоваться внешняя синхронизация или атомарные методы (`AddOrUpdate`).
    
- Избегайте частых преобразований коллекции в массив (`ToArray`) в горячих сценариях — это создаёт дополнительную аллокацию и копирование.
    
- Для случаев, когда нужен строгий порядок (например, очередность по приоритету), рассмотрите `ConcurrentPriorityQueue` из сторонних библиотек или свои реализации.
    
- Всегда выбирайте коллекцию с наиболее подходящими гарантиями порядка и производительности:
    
    - FIFO — `ConcurrentQueue`/`BlockingCollection`
        
    - LIFO — `ConcurrentStack`
        
    - Неупорядоченно, high-throughput — `ConcurrentBag`
        

---

Эти коллекции позволяют писать высокоэффективный многопоточный код без явных блокировок, снижая сложность и риск ошибок. ```
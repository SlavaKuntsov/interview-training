Коллекции в .NET — это классы, которые предоставляют способы хранения, управления и доступа к группам объектов. В .NET существует несколько семейств коллекций, каждая из которых предназначена для определённых сценариев использования: работа с массивами, списками, словарями, множествами, очередями, стеками и т.д.

## Основные группы коллекций

.NET предлагает коллекции из следующих пространств имён:

- `System.Collections` — базовые коллекции (несовременные, non-generic).
- `System.Collections.Generic` — обобщённые коллекции (рекомендуемые к использованию).
- `System.Collections.Concurrent` — потокобезопасные коллекции.
- `System.Collections.Immutable` — неизменяемые коллекции.

---

## 1. Обобщённые коллекции (`System.Collections.Generic`)

Используются чаще всего, так как обеспечивают типобезопасность и производительность без boxing/unboxing.

### List<T>
Динамический массив.

```csharp
List<int> numbers = new List<int> { 1, 2, 3 };
numbers.Add(4);
numbers.RemoveAt(0);
````

- Быстрый доступ по индексу: `O(1)`
    
- Добавление в конец: `Amortized O(1)`
    
- Удаление из середины: `O(n)` (сдвиг элементов)
    

### Dictionary<TKey, TValue>

Хеш-таблица. Быстрый доступ по ключу.

```csharp
Dictionary<string, int> ages = new();
ages["Alice"] = 30;
int age = ages["Alice"];
```

- Доступ по ключу: `O(1)` при хорошей хеш-функции
    
- Ключи должны быть уникальны
    

### HashSet

Множество без повторяющихся элементов.

```csharp
HashSet<string> set = new() { "a", "b" };
set.Add("c");  // добавит
set.Add("a");  // не добавит
```

- Используется для быстрой проверки принадлежности: `O(1)`
    

### Queue и Stack

- `Queue<T>` — FIFO:
    
    ```csharp
    Queue<int> q = new();
    q.Enqueue(1);
    var x = q.Dequeue();  // 1
    ```
    
- `Stack<T>` — LIFO:
    
    ```csharp
    Stack<string> s = new();
    s.Push("a");
    var val = s.Pop();  // "a"
    ```
    

---

## 2. Необобщённые коллекции (`System.Collections`)

Исторически первые коллекции. Не используют обобщения, работают через `object`, что приводит к:

- Потере типобезопасности
    
- Boxing/unboxing для value-types
    
- Снижению производительности
    

Примеры:

- `ArrayList`: динамический список (`List<object>`)
    
- `Hashtable`: хеш-таблица (`Dictionary<object, object>`)
    

Не рекомендуется использовать в новом коде.

---

## 3. Потокобезопасные коллекции (`System.Collections.Concurrent`)

Используются при работе с несколькими потоками.

### ConcurrentDictionary<TKey, TValue>

```csharp
ConcurrentDictionary<string, int> dict = new();
dict.TryAdd("a", 1);
dict.TryGetValue("a", out int val);
```

- Высокопроизводительная замена `Dictionary` для многопоточности.
    
- Разделение блокировок по сегментам.
    

### BlockingCollection

- Обёртка над очередью с поддержкой блокировки и ожидания.
    
- Применяется в Producer-Consumer сценариях.
    

```csharp
var coll = new BlockingCollection<int>();
Task.Run(() => coll.Add(1));
int x = coll.Take(); // блокирует, если пусто
```

---

## 4. Неизменяемые коллекции (`System.Collections.Immutable`)

Позволяют безопасно передавать данные между потоками без блокировок.

```csharp
var immutableList = ImmutableList<int>.Empty;
var updated = immutableList.Add(1).Add(2);
```

- Каждое изменение возвращает новую версию структуры.
    
- Используются в функциональном программировании.
    

---

## 5. Span, Memory (специальные структуры для буферов)

`Span<T>` — неколлекция, но структура для доступа к непрерывному блоку памяти:

```csharp
Span<int> numbers = stackalloc int[3] { 1, 2, 3 };
numbers[0] = 10;
```

- Не аллоцирует в куче
    
- Используется в high-performance сценариях
    

---

## Выбор коллекции

|Сценарий|Коллекция|
|---|---|
|Быстрый доступ по индексу|`List<T>`, `Array`|
|Быстрый доступ по ключу|`Dictionary<TKey, TValue>`|
|Без повторов|`HashSet<T>`|
|LIFO|`Stack<T>`|
|FIFO|`Queue<T>`|
|Thread-safe словарь|`ConcurrentDictionary`|
|Передача между потоками (блокировки)|`BlockingCollection`|
|Многопоточная безопасная передача|`ImmutableList`, `ImmutableDictionary`|
|Управление буферами без GC-алокаций|`Span<T>`, `Memory<T>`|

---

## Особенности и подводные камни

- Коллекции не thread-safe по умолчанию (кроме Concurrent/Immutable).
    
- `List<T>` увеличивает внутренний буфер экспоненциально (`Capacity * 2`), это может влиять на память.
    
- При частом удалении/вставке в середину — лучше использовать `LinkedList<T>`.
    
- `Dictionary` может «раздуваться» при удалении большого числа элементов — стоит вызвать `TrimExcess()`.
    

---

## LINQ и коллекции

LINQ (Language Integrated Query) предоставляет удобные методы работы с коллекциями:

```csharp
var evens = list.Where(x => x % 2 == 0).ToList();
```

- Все методы LINQ работают с `IEnumerable<T>`, а значит — с любой коллекцией.
    
- Не забывайте: `ToList()`, `ToArray()` вызывают аллокацию.
    
- Отложенное выполнение (`Where`, `Select`) требует осторожности при множественных проходах.
    

---

## IEnumerable, ICollection, IList

### Интерфейсы

- `IEnumerable<T>` — только перечисление (`foreach`)
    
- `ICollection<T>` — добавление/удаление элементов
    
- `IList<T>` — доступ по индексу
    

Например:

```csharp
List<int> list = new();
IEnumerable<int> e = list;
ICollection<int> c = list;
IList<int> il = list;
```

Понимание интерфейсов важно для ограничения доступа и соблюдения принципа минимального интерфейса.

---

## Заключение

Коллекции — фундамент любого приложения на .NET. Для производительности, читаемости и правильной архитектуры важно:

- Осознанно выбирать коллекции под задачу
    
- Понимать сложность операций
    
- Избегать лишних аллокаций и копирований
    
- Правильно работать с потоками и памятью
    

```csharp
// Пример: безопасное создание уникального набора
var uniqueEmails = new HashSet<string>(
    users.Select(u => u.Email),
    StringComparer.OrdinalIgnoreCase);
```
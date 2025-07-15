### Концепция и назначение  
`Span<T>` и `Memory<T>` — новые примитивы для работы с непрерывными участками памяти без аллокаций в куче, введённые в .NET Core и C# 7.2+. Они позволяют обрабатывать массивы, буферы стековой, управляемой или неуправляемой памяти единообразно и безопасно.

---

### Span\<T>

- **Структура**: `ref struct Span<T>` — **значимый** тип, живущий только в стеке.  
- **Использование**: обеспечивает **быстрый** доступ к последовательности элементов; может указывать на:
  - элементы массива (`T[]`),
  - сегмент управляемой кучи (`new T[N]`),
  - стековый буфер через `stackalloc`,
  - неуправляемую память через `MemoryMarshal`.
- **Ограничения**:
  - Не может быть полем класса или захватываться лямбдой — живёт только в методе/блоке.
  - Не может вызываться асинхронно (нельзя `await`), не может передаваться в другой поток.
- **API**:
  - `Span<T> slice = original.Slice(start, length);`
  - методы `Clear()`, `Fill(value)`, `CopyTo(destination)`, `TryCopyTo(destination)`.
  - индексатор `span[i]`, свойства `Length`.
- **Пример**:

```csharp
  void ReverseBuffer(Span<int> span)
  {
      int i = 0, j = span.Length - 1;
      while (i < j)
      {
          int tmp = span[i];
          span[i++] = span[j];
          span[j--] = tmp;
      }
  }
  // Вызов с разными источниками:
  var arr = new int[] {1,2,3,4};
  ReverseBuffer(arr);                     // из массива
  Span<int> stack = stackalloc int[4];    
  ReverseBuffer(stack);                   // из стека
```


---


### Memory<T> и ReadOnlyMemory<T>


- **Структура**: `struct Memory<T>` и `struct ReadOnlyMemory<T>` — **значимые** типы, сохраняющие ссылку на память **в куче**, могут быть полями и возвращаться из методов.
    
- **Различие от `Span<T>`**:
    
    - **Время жизни**: живут в управляемой куче, не привязаны к стеку.
        
    - **Асинхронность**: могут передаваться между методами, храниться в объектах, используются в сочетании с `await`.
        
    - **Атрибуты**: отличаются только поддержкой мутабельности (`ReadOnlyMemory<T>`).
        
- **Преобразование**:
    
    - `Span<T> span = memory.Span;`
        
    - `ReadOnlySpan<T> readOnlySpan = readOnlyMemory.Span;`
        
- **API**:
    
    - `Memory<T>.Slice(start, length)`, `ReadOnlyMemory<T>.Slice(...)`.
        
    - Асинхронная обработка: `var writer = new PipeWriter(memory);` и другие I/O API принимают `Memory<byte>`.
        
- **Пример**:
    
    ```csharp
    class BufferHolder
    {
        private Memory<byte> _buffer;
        public BufferHolder(int size) => _buffer = new byte[size];
        public async Task FillAsync(Stream stream, CancellationToken ct)
        {
            int read = await stream.ReadAsync(_buffer, ct); // асинхронно читает в Memory<byte>
            Process(_buffer.Slice(0, read).Span);
        }
        private void Process(Span<byte> data) { /* … */ }
    }
    ```
    

---

### Производительность и сценарии

- **Минимизация GC-давления**: никакие промежуточные массивы или аллокации для срезов.
    
- **Локальность кэша**: работа с `Span<T>`/`Memory<T>` повышает шансы попадания в L1/L2 кэш.
    
- **Безопасность**: проверка границ индексация выбросит `IndexOutOfRangeException`, избавляя от UB.
    

**Когда использовать**:

- Горячие участки кода, где важно избежать аллокаций (парсеры, сериализация, криптооперации).
    
- Асинхронные I/O операции (`Stream.ReadAsync`, `PipeWriter.WriteAsync`) с `Memory<byte>`.
    
- Работа с партией данных из разных источников — единый API для массива, строки, пиринговых буферов.
    

---

### Особые приёмы

- **`stackalloc` + `Span<T>`** для небольших временных буферов (< 1KB).
    
- **`MemoryPool<T>`** из `System.Buffers` для аренды больших буферов без аллокаций массива.
    
```csharp
    using var owner = MemoryPool<byte>.Shared.Rent(1024);
    Memory<byte> buffer = owner.Memory.Slice(0, 512);
```
    
- **`Span<T>` над строками**: `ReadOnlySpan<char> text = myString.AsSpan();` для быстрого поиска и разбора.
    

---

Грамотное применение `Span<T>` и `Memory<T>` позволяет создавать эффективные, низколатентные решения в .NET, сократить количество аллокаций и нагрузку на GC, сохраняя при этом безопасность и читаемость кода
 **Пайплайн обработки запроса:**

1. **LINQ-запрос (C#)**
    
    - Разрабатывайте запрос, используя `DbSet<TEntity>` и методы расширения `IQueryable<T>`, например:
        
        ```
        var query = context.Products
            .Where(p => p.Price > 100)
            .OrderBy(p => p.Name)
            .Select(p => new { p.Id, p.Name, p.Price });
        ```
        
2. **Построение Expression Tree**
    
    - Компилятор C# создает дерево выражений (`Expression<Func<TEntity, bool>>` и т.д.)
        
    - Каждый вызов методов расширения (`Where`, `OrderBy`, `Select`) добавляет узел в это дерево.
        
3. **Передача в IQueryProvider**
    
    - `DbSet<TEntity>` реализует `IQueryable<TEntity>`, а под капотом хранится `QueryProvider` от EF Core.
        
    - При первом же вызове терминального метода (например, `ToList()`, `FirstOrDefault()`, `CountAsync()`) начинается выполнение.
        
4. **Преобразование Expression Tree в SQL**
    
    - **Expression Visitor** разбирает дерево выражений.
        
    - Проводится **трансляция** узлов в SQL-конструкции (`WHERE`, `ORDER BY`, `SELECT`, `JOIN` и т.д.).
        
    - **Модификаторы (Query Modifiers)** влияют на генерируемый SQL:
        
        - **AsNoTracking()**: убирает отслеживание сущностей, убирает `LEFT JOIN` с таблицей `__EFMigrationsHistory`
            
        - **AsTracking()**: возвращает поведение по умолчанию (отслеживание изменений)
            
        - **AsSplitQuery() / AsSingleQuery()**: контролируют объединение `Include`-запросов в один SQL или разделение на несколько
            
        - **TagWith("text")**: добавляет комментарий в SQL, полезно для трассировки и отладки
            
    - Пример:
        
        ```
        var result = context.Orders
            .TagWith("Получаем заказы за вчера")
            .Where(o => o.OrderDate >= yesterday)
            .AsNoTracking()
            .ToList();
        ```
        
5. **Выполнение SQL**
    
    - EF Core использует ADO.NET: открывает соединение, создает `DbCommand`, добавляет **параметры** (SQL injection защищено) и вызывает `ExecuteReader()`.
        
    - Результат — `DbDataReader`.
        
6. **Materialization (материализация объектов)**
    
    - **Object Materializer** читает каждую строку `DbDataReader`.
        
    - На основе метаданных сопоставляются поля из запроса и свойства сущности.
        
    - Создаются объекты сущностей или проекций (анонимных или DTO).
        
    - Если включено отслеживание, сущности сохраняются в `ChangeTracker`.
        

**Терминальные методы (Trigger Execution):**

- `ToList()`, `ToArray()`, `Single()`, `First()`, `Count()`, `Any()`, `Sum()`, `Max()` и их асинхронные аналоги вызывают запрос к базе.
    
- До вызова этих методов запрос является **отложенным** (deferred execution).
    

**Модификаторы и операторы LINQ:**

- **Where(predicate)** — добавляет `WHERE`
    
- **OrderBy / OrderByDescending** — добавляют `ORDER BY`
    
- **Skip(count), Take(count)** — `OFFSET ... FETCH` (пагинация)
    
- **GroupBy(keySelector)** — `GROUP BY`
    
- **Join / GroupJoin** — `INNER JOIN`, `LEFT JOIN`
    
- **Select(selector)** — формирует список столбцов в `SELECT`
    
- **Include / ThenInclude** — `JOIN` или дополнительные запросы (зависит от `AsSplitQuery`)
    
- **Distinct** — `SELECT DISTINCT`
    

**Параметры и SQL Injection**

- При трансляции EF Core генерирует `@__p0`, `@__p1` параметры, значения которых передаются через `DbParameter`.
    
- Это полностью исключает риски SQL-инъекций.
    

**Оптимизация и кэширование**

- **Compiled Queries** (`EF.CompileQuery`) кешируют скомпилированный план выполнения LINQ → IL, ускоряя повторные вызовы.
    
- **Query cache** хранит метаинформацию и планы SQL в памяти на протяжении жизни `DbContext`.
    

**Логи и профилирование:**

- Настройка логирования в `DbContextOptionsBuilder`
    
    ```
    optionsBuilder
        .EnableSensitiveDataLogging()
        .LogTo(Console.WriteLine, LogLevel.Information);
    ```
    
- Инструменты: **MiniProfiler**, **EFCoreSecondLevelCacheInterceptor**, **SqlServer Profiler**
    

**Пакеты и расширения:**

- `Microsoft.EntityFrameworkCore.Relational` — содержит API для реляционных баз (SQL, JOIN, транзакции)
    
- `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`, `Pomelo.EntityFrameworkCore.MySql` — провайдеры баз данных
    
- `Z.EntityFramework.Plus` — добавляет расширения запросов (BatchDelete, DeferredJoin)
    
- `EntityFrameworkCore.Triggers` — хуки до/после операций
    

---

Теперь у тебя есть глубокое понимание того, как EF Core обрабатывает запросы: от составления LINQ-выражения до материализации объектов, включая ключевые модификаторы и возможности оптимизации.

`using` служит для безопасного и автоматического освобождения ресурсов, реализующих интерфейс `IDisposable`.

## Основные формы

### 1. Using-statement (блок)

```csharp
using (var resource = new StreamReader(path))
{
    // Работа с resource
} // Здесь автоматически вызывается resource.Dispose()
```

- Компилятор разворачивает в `try-finally`:
    
    ```csharp
    var resource = new StreamReader(path);
    try
    {
        // ...
    }
    finally
    {
        if (resource != null)
            resource.Dispose();
    }
    ```
    

### 2. Using-declaration (C# 8.0+)

```csharp
using var conn = new SqlConnection(connString);
// ...использование conn...
// Dispose будет вызван при выходе из текущей области видимости
```

- Не требует дополнительного блока `{}`.
    
- Удобно для нескольких ресурсов подряд:
    
    ```csharp
    using var file = File.OpenRead(filePath);
    using var reader = new StreamReader(file);
    // Логика...
    ```
    

## Детали реализации

- `using` применим только к объектам, реализующим `IDisposable`.
    
- Внутри компилятор генерирует вызов `Dispose()`, что позволяет избежать утечек ресурсов.
    
- В .NET `Dispose()` освобождает управляемые и/или неуправляемые ресурсы (файлы, сокеты, соединения к БД).
    

## ConfigureAwait(false)

В асинхронном коде часто используют:

```csharp
await someTask.ConfigureAwait(false);
```

- Отключает возвращение в исходный `SynchronizationContext`.
    
- Полезно в серверных или библиотечных методах для повышения производительности и предотвращения deadlock.
    

## Паттерны и лучшие практики

- **Всегда** освобождайте `IDisposable`-ресурсы: потоки, сокеты, соединения, объекты COM.
    
- **Using-statement** подходит для вложенных операций с ресурсами.
    
- **Using-declaration** — более компактный код, особенно при нескольких ресурсах.
    
- В GUI/ASP.NET Core-коде при асинхронных операциях комбинируйте `await` и `using`:
    
    ```csharp
    await using var fs = new FileStream(path, FileMode.Open);
    // C# 8: IAsyncDisposable
    await fs.FlushAsync();
    ```
    

## IAsyncDisposable и await using (C# 8.0+)

- Для асинхронного освобождения ресурсов: интерфейс `IAsyncDisposable`.
    
- Синтаксис:
    
    ```csharp
    await using var resource = new AsyncResource();
    ```
    
- Аналогично генерируется `try-finally` с `await resource.DisposeAsync()`.
    

## Отличия и сравнение

|Форма|Область видимости|Автоматический try/finally|Асинхронный Dispose|
|---|---|---|---|
|using-statement|Блок `{}`|Да|Нет|
|using-declaration|Метод/свойство/блок|Да|Нет|
|await using|Метод async|Да|Да|

## Заключение

Корректное использование `using` гарантирует своевременное освобождение ресурсов и повышает надёжность приложений на .NET.
Фильтры — это механизм расширения конвейера обработки запросов в MVC и минимальных API. Они позволяют выполнять код до и после выполнения действий контроллеров (actions), управлять доступом, кэшированием, обработкой исключений и результатами. ASP.NET Core поддерживает несколько типов фильтров, каждый из которых срабатывает на своем этапе обработки.

## Типы фильтров и порядок их выполнения

1. **Authorization Filters**  
   Выполняются на самом раннем этапе. Используются для проверки прав доступа и аутентификации.  
2. **Resource Filters**  
   Обёртывают весь жизненный цикл работы с ресурсом: выполняются до и после выполнения кэширования, model binding и фильтров других типов.  
3. **Model Binding**  
   Привязка данных из запроса к параметрам action и модели.  
4. **Action Filters**  
   Выполняются до и после выполнения метода action.  
5. **Exception Filters**  
   Срабатывают при возникновении необработанных исключений в action или в других фильтрах ниже по уровню.  
6. **Result Filters**  
   Выполняются до и после формирования `IActionResult`, но до записи ответа в тело HTTP.  

Порядок:  
```

Authorization → Resource (OnResourceExecuting) → Model Binding → Action (OnActionExecuting) → Action Execution → Action (OnActionExecuted) →  
Exception (если исключение) → Result (OnResultExecuting) → Result Execution → Result (OnResultExecuted) → Resource (OnResourceExecuted)

````

## Authorization Filters

- Реализуют интерфейс `IAuthorizationFilter` или `IAsyncAuthorizationFilter`.  
- Чаще всего используются атрибуты `[Authorize]`, `[AllowAnonymous]`.  
- В `OnAuthorization` можно вручную проверять `HttpContext.User` и устанавливать `context.Result = new ForbidResult()` для отказа в доступе.

```csharp
public class CustomAuthFilter : IAsyncAuthorizationFilter
{
    public async Task OnAuthorizationAsync(AuthorizationFilterContext context)
    {
        var user = context.HttpContext.User;
        if (!user.Identity.IsAuthenticated || !user.IsInRole("Admin"))
            context.Result = new ForbidResult();
    }
}
````

## Resource Filters

- Реализуют `IResourceFilter` или `IAsyncResourceFilter`.
    
- Срабатывают до model binding и после выполнения всех последующих фильтров и action.
    
- Подходят для кэширования (`ResponseCaching`), открытия/закрытия транзакций, логирования запроса.
    

```csharp
public class TransactionFilter : IAsyncResourceFilter
{
    private readonly IDbContextFactory<MyDbContext> _factory;

    public TransactionFilter(IDbContextFactory<MyDbContext> factory)  
        => _factory = factory;

    public async Task OnResourceExecutionAsync(ResourceExecutingContext context, ResourceExecutionDelegate next)
    {
        await using var db = _factory.CreateDbContext();
        await using var transaction = await db.Database.BeginTransactionAsync();

        var executedContext = await next(); // выполнение action и других фильтров

        if (executedContext.Exception == null)
            await transaction.CommitAsync();
        else
            await transaction.RollbackAsync();
    }
}
```

## Action Filters

- Реализуют `IActionFilter` или `IAsyncActionFilter`.
    
- Срабатывают непосредственно до и после метода action.
    
- Используются для валидации модели, логирования входных параметров, измерения времени выполнения.
    

```csharp
public class LoggingActionFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var stopwatch = Stopwatch.StartNew();
        var executed = await next(); // запуск action
        stopwatch.Stop();

        var actionName = context.ActionDescriptor.DisplayName;
        var elapsed = stopwatch.Elapsed;
        // Логируем actionName и elapsed
    }
}
```

## Exception Filters

- Реализуют `IExceptionFilter` или `IAsyncExceptionFilter`.
    
- Срабатывают при необработанных исключениях в action или предыдущих фильтрах.
    
- Могут формировать единый формат ответов об ошибках, логировать, выполнять перекрытие исключений.
    

```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        // Логирование context.Exception
        context.Result = new ObjectResult(new { error = "Internal server error" })
        { StatusCode = StatusCodes.Status500InternalServerError };
        context.ExceptionHandled = true;
    }
}
```

## Result Filters

- Реализуют `IResultFilter` или `IAsyncResultFilter`.
    
- Выполняются до и после `IActionResult.ExecuteResultAsync()`.
    
- Позволяют изменять результат перед отправкой в ответ (например, оборачивать данные в единый формат).
    

```csharp
public class WrapResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        if (context.Result is ObjectResult obj)
        {
            var original = obj.Value;
            obj.Value = new { data = original, timestamp = DateTime.UtcNow };
        }
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}
```

## Регистрация и приоритет

- Фильтры можно применять:
    
    - Глобально в `Startup.ConfigureServices`:
        
        ```csharp
        services.AddControllers(options =>
        {
            options.Filters.Add<LoggingActionFilter>();
        });
        ```
        
    - На контроллере или action через атрибут:
        
        ```csharp
        [ServiceFilter(typeof(CustomAuthFilter))]
        public class AdminController : ControllerBase { … }
        ```
        
    - Через `TypeFilterAttribute` и `ServiceFilterAttribute` для внедрения зависимостей.
        
- Приоритет выполнения определяется порядком добавления и типом фильтра. Глобальные фильтры обычно срабатывают до атрибутных.
    

## Различия между синхронными и асинхронными фильтрами

- Используйте асинхронные версии (`IAsync*`) для неblocking операций (I/O, база данных).
    
- Синхронные блокируют поток и могут снижать масштабируемость.
    

## Когда использовать

- **Authorization Filters** — проверка прав доступа до любой работы.
    
- **Resource Filters** — транзакции, кэширование, логирование запросов.
    
- **Action Filters** — валидация, логирование параметров, измерение времени.
    
- **Exception Filters** — глобальная обработка ошибок.
    
- **Result Filters** — форматирование и обёртка результатов.
    

---

_Материал основан на официальной документации ASP.NET Core и практике создания расширяемых веб-приложений._
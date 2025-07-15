>  https://stefaniuk.website/all/zhiznenny-cikl-zaprosov-v-asp-net-core-mvc/

Когда ASP.NET Core приложение получает HTTP‑запрос, он проходит через несколько этапов и компонентов, которые формируют единый конвейер (pipeline). Ниже приведено подробное описание ключевых шагов и компонентов.

## 1. Запуск и хостинг

1. **Program.cs / Generic Host**  
   - Создаётся `HostBuilder` или `WebApplicationBuilder`.  
   - Конфигурируются настройки (`Configuration`), сервисы (`ConfigureServices`) и конвейер (`Configure`).  
   - Запускается хост: `host.Run()` или `app.Run()`.

2. **Встраиваемый веб‑сервер (Kestrel / IIS / HTTP.sys)**  
   - Kestrel — кроссплатформенный, легковесный, запускается внутри процесса.  
   - При интеграции с IIS используется IIS‑HTTP Platform Handler.  
   - HTTP.sys для Windows‑specific сценариев.

## 2. Приём запроса

1. **Network Listener** (Kestrel)  
   - Слушает указанный порт и IP.  
   - При поступлении TCP‑пакета собирает полный HTTP‑запрос (с учётом заголовков, тела).

2. **HTTP Parser**  
   - Преобразует байты в объект `HttpContext.Request`:
     - `Method`, `Scheme`, `Host`, `Path`, `QueryString`.  
     - Заголовки (`IHeaderDictionary`).  
     - Тело (`HttpRequest.Body` — поток).

## 3. Middleware Pipeline

В `Configure(IApplicationBuilder app)` регистрируются посредники (middleware). Они выполняются строго в порядке регистрации.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseMiddleware<ErrorHandlingMiddleware>();
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
````

Каждый middleware получает `HttpContext` и функцию `next()` для передачи управления дальше:

```csharp
public async Task InvokeAsync(HttpContext context, RequestDelegate next)
{
    // Действия до следующих компонентов
    await next(context);
    // Действия после следующих компонентов
}
```

### Основные Middleware

- **Exception Handling** (`UseExceptionHandler`, `DeveloperExceptionPage`)
    
- **HTTPS Redirection** (`UseHttpsRedirection`)
    
- **Static Files** (`UseStaticFiles`)
    
- **Routing** (`UseRouting`)
    
- **CORS** (`UseCors`)
    
- **Authentication** (`UseAuthentication`)
    
- **Authorization** (`UseAuthorization`)
    
- **Endpoints** (`UseEndpoints` / `MapControllers`, `MapRazorPages`, `MapGet` для минимальных API)
    

## 4. Маршрутизация (Routing)

1. **Endpoint Routing**
    
    - `UseRouting` строит карту (route table) из зарегистрированных конечных точек (endpoints).
        
    - По пути и HTTP‑методу выбирает подходящую конечную точку (`RouteEndpoint`).
        
2. **Выбор конечной точки**
    
    - Параметры маршрута (route parameters) и их привязка.
        
    - Constraints (ограничения по шаблону, HTTP‑методу).
        
3. **Вызов Middleware Authorization / CORS**
    
    - После `UseRouting` и до `UseEndpoints` могут выполняться проверки политики CORS и авторизации.
        

## 5. Обработка в MVC / Minimal API

### MVC Controllers

1. **Model Binding**
    
    - Извлечение данных из маршрута, query‑string, тела (JSON/XML), форм, заголовков в параметры методов контроллера и свойства модели.
        
2. **Фильтры**
    
    - **Authorization Filters**
        
    - **Resource Filters**
        
    - **Action Filters**
        
    - **Exception Filters**
        
    - **Result Filters**  
        Выполняются до и после вызова действия (action).
        
3. **Action Execution**
    
    - Вызов метода контроллера (`ControllerBase` или `Controller`).
        
    - Возвращённый объект (`IActionResult`, POCO) передаётся дальше.
        
4. **Result Execution**
    
    - Обёртывание результата (`ObjectResult`, `JsonResult`, `ViewResult`) в HTTP‑ответ: установка заголовков, сериализация тела.
        

### Минимальные API

```csharp
var app = WebApplication.CreateBuilder(args).Build();
app.MapGet("/weather/{city}", (string city) => GetForecast(city));
app.Run();
```

- Без контроллеров и фильтров: маршрут напрямую связывается с делегатом.
    
- Data binding и результат сериализуется автоматически.
    

## 6. Формирование и отправка ответа

1. **HttpResponse**
    
    - Установка `StatusCode`, `Headers`.
        
    - Запись в поток тела ответа (`HttpResponse.Body`).
        
    - Поддержка push‑stream, file‑stream, chunked encoding.
        
2. **Logging / Metrics**
    
    - После отправки ответа могут срабатывать middleware логирования (`Serilog`, `App.Metrics`).
        
3. **Connection Teardown**
    
    - Закрытие или удержание соединения (Keep-Alive).
        
    - Освобождение ресурсов `HttpContext`.
        

---

**Ключевые принципы:**

- Pipeline строится из отдельных, переиспользуемых middleware.
    
- Чёткая последовательность: приём → middleware → routing → action → result → ответ.
    
- Расширяемость: свои middleware, фильтры, endpoint conventions.
    
- Высокая производительность благодаря минимальным оверхедам и асинхронной модели.
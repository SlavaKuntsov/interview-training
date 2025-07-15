### Архитектура и жизненный цикл запроса

1. **HTTP запрос** поступает в веб‑сервер (Kestrel/IIS/Nginx).  
2. **Middleware pipeline** в `Startup.Configure` последовательно обрабатывает запросы (например, логирование, статические файлы, авторизация).  
3. **Routing** (`UseRouting`) сопоставляет URL-шаблон с маршрутом контроллера.  
4. **Endpoint execution** (`UseEndpoints`) вызывает MVC‑Middleware, который:
   - Создаёт **ControllerContext**
   - Находит нужный **Controller** и **Action** метод
   - Выполняет **model binding** (привязывает параметры из route/query/body к аргументам метода)
   - Запускает **ActionFilters** и **AuthorizationFilters**
   - Вызывает сам **Action** метод контроллера
   - Обрабатывает возвращённый `IActionResult` (ViewResult, JsonResult, Redirect, StatusCode и т. д.)
5. **Рендеринг View** (если применимо) с Razor‑движком  
6. **Ответ** отправляется обратно через pipeline в веб‑сервер и клиенту

---

### Основные концепции

#### Контроллеры и действия
- Контроллер — класс, наследующий от `Controller` или помеченный `[ApiController]` для API.  
- Action — публичный метод контроллера, возвращающий `IActionResult` или конкретный тип (при `[ApiController]` автоматически оборачивается в `Ok()`, `BadRequest()` и т. д.).  
- Атрибуты маршрутизации:  
```csharp
  [Route("products")]
  public class ProductsController : Controller
  {
      [HttpGet("")]
      public IActionResult Index() { ... }

      [HttpGet("{id:int}")]
      public IActionResult Details(int id) { ... }

      [HttpPost("create")]
      public IActionResult Create(ProductModel model) { ... }
  }
```

#### Модели и привязка (Model Binding)

- **ViewModel** / **DTO** — классы, описывающие данные формы или JSON‑запроса.
    
- Model Binding автоматически наполняет свойства из:
    
    - Route parameters
        
    - Query string
        
    - Form data (`application/x-www-form-urlencoded`)
        
    - Тело запроса (`application/json`, `multipart/form-data`)
        
- **Data Annotations** для валидации:
    
    ```csharp
    public class ProductModel
    {
        [Required]
        public string Name { get; set; }
    
        [Range(0.01, 10000)]
        public decimal Price { get; set; }
    }
    ```
    
- В контроллере проверка:
    
    ```csharp
    if (!ModelState.IsValid)
        return View(model);
    ```
    

#### Представления (Views) и Razor

- Расширение `.cshtml`, микс C# (@) и HTML.
    
- **Layout** (`_Layout.cshtml`) задаёт общий каркас страницы: `<head>`, `<body>`, секцию `@RenderBody()`.
    
- **Partial Views** (`_ProductItem.cshtml`) для переиспользуемых фрагментов.
    
- **Tag Helpers** для безопасного и выразительного HTML:
    
    ```html
    <form asp-controller="Products" asp-action="Create" method="post">
        <input asp-for="Name" />
        <span asp-validation-for="Name"></span>
        <button type="submit">Create</button>
    </form>
    ```
    
- **View Components** для сложных переиспользуемых UI‑фрагментов с собственным кодом.
    

#### Dependency Injection

- По умолчанию в `Startup.ConfigureServices`:
    
    ```csharp
    services.AddControllersWithViews();
    services.AddScoped<IProductRepository, SqlProductRepository>();
    services.AddDbContext<AppDbContext>(opts => opts.UseSqlServer(conn));
    ```
    
- Внедрение через конструктор:
    
    ```csharp
    public class ProductsController : Controller
    {
        private readonly IProductRepository _repo;
        public ProductsController(IProductRepository repo) => _repo = repo;
    }
    ```
    

#### Фильтры (Filters)

- **AuthorizationFilter** (`[Authorize]`)
    
- **ResourceFilter** (кэширование, логи)
    
- **ActionFilter** (`OnActionExecuting`, `OnActionExecuted`)
    
- **ExceptionFilter** (глобальная обработка ошибок)
    
- **ResultFilter** (действия перед/после выполнения result)
    
- Регистрация глобально в `ConfigureServices`:
    
    ```csharp
    services.AddControllersWithViews(options =>
    {
        options.Filters.Add<CustomExceptionFilter>();
    });
    ```
    

#### Статические файлы, сессии, кэширование

- `app.UseStaticFiles()` для выдачи `wwwroot`.
    
- `services.AddResponseCaching()` + `app.UseResponseCaching()` для server-side кэширования.
    
- `services.AddDistributedMemoryCache()` + `services.AddSession()` + `app.UseSession()` для хранения пользовательских данных между запросами.
    

---

### Разработка API с MVC

- Пометка контроллера `[ApiController]`:
    
    - Автоматическая привязка тела запроса к `FromBody`
        
    - `400 BadRequest` при невалидной модели
        
    - Атрибуты `[FromQuery]`, `[FromRoute]`, `[FromForm]` для явного указания источников данных
        
- Возврат конкретных типов:
    
    ```csharp
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsApiController : ControllerBase
    {
        [HttpGet]
        public ActionResult<IEnumerable<ProductDto>> GetAll() => Ok(_repo.GetAll());
    }
    ```
    
- Поддержка OpenAPI/Swagger:
    
    ```csharp
    services.AddSwaggerGen();
    app.UseSwagger();
    app.UseSwaggerUI();
    ```
    

---

### Практические советы

- Разделение **слоёв**: контроллеры, сервисы (бизнес-логика), репозитории, модели.
    
- Используйте **ViewModels** вместо сущностей EF Core в представлениях.
    
- Включайте **аннотации и FluentValidation** для валидации входных данных.
    
- Логируйте запросы и исключения через **Serilog**, **ILogger<T>**.
    
- Настройте **Глобальную обработку ошибок** и дружелюбные страницы 404/500.
    
- Оптимизируйте выдачу статических ресурсов, минификацию и бандлинг (вспомогательные пакеты).
    

ASP.NET Core MVC — гибкая, модульная платформа для создания веб‑приложений и API, сочетающая проверенные паттерны MVC с современными возможностями .NET и высокопроизводительным web‑сервером.```
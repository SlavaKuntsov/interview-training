 **Dependency Injection (DI)** — это шаблон проектирования, который помогает управлять зависимостями в приложении, передавая необходимые зависимости извне. В ASP.NET Core DI встроен в фреймворк и позволяет автоматически управлять зависимостями, устраняя необходимость вручную создавать объекты и управлять их временем жизни.

### Основные типы DI в ASP.NET Core

В ASP.NET Core DI-контейнер поддерживает три типа времени жизни зависимостей: **AddTransient**, **AddScoped**, и **AddSingleton**. Они определяют, как долго объект остается в памяти и как часто создается новый экземпляр.

---

### 1. **AddTransient**

- **Описание**: Регистрация зависимости с помощью `AddTransient` означает, что каждый раз при обращении к ней будет создаваться новый объект.
- **Сценарии использования**: Этот подход подходит для легковесных, краткоживущих объектов, которые не сохраняют состояние между запросами. Например, для вспомогательных сервисов, работающих с данными запроса, или утилит.
- **Преимущества**: 
  - Объекты не сохраняют состояние, что делает их более безопасными для использования в многопоточной среде, так как нет риска использования одного и того же объекта разными потоками.
- **Пример**:
  ```csharp
  public class MyTransientService : IMyService
  {
      public void PerformAction() 
      {
          // Логика сервиса
      }
  }

  // Регистрация сервиса
  services.AddTransient<IMyService, MyTransientService>();
  ```

  При каждом вызове `IMyService` создается новый экземпляр `MyTransientService`.

---

### 2. **AddScoped**

- **Описание**: При регистрации зависимости с помощью `AddScoped` объект создается один раз для каждого HTTP-запроса. В рамках одного запроса все компоненты, использующие эту зависимость, будут получать один и тот же экземпляр.
- **Сценарии использования**: Подходит для сервисов, которые должны сохранять состояние в пределах одного запроса, но не между запросами. Например, для работы с данными запроса, пользовательскими данными или обработкой транзакций.
- **Преимущества**:
  - Избегается создание новых объектов при каждом обращении в рамках одного запроса, что повышает производительность.
  - Удобно для работы с транзакциями в рамках одного запроса.
- **Пример**:
  ```csharp
  public class MyScopedService : IMyService
  {
      public void PerformAction()
      {
          // Логика сервиса
      }
  }

  // Регистрация сервиса
  services.AddScoped<IMyService, MyScopedService>();
  ```

  В течение одного HTTP-запроса при каждом обращении к `IMyService` будет возвращаться один и тот же экземпляр `MyScopedService`.

---

### 3. **AddSingleton**

- **Описание**: Регистрация зависимости с помощью `AddSingleton` означает, что объект создается один раз при первом обращении и остается в памяти на протяжении всего жизненного цикла приложения. Все запросы к этой зависимости возвращают один и тот же экземпляр.
- **Сценарии использования**: Singleton-подход подходит для объектов, которые содержат кэшируемые данные или не изменяемое состояние, либо для объектов, инициализация которых занимает много ресурсов. Пример: конфигурация приложения, кэш или сервисы, работающие с внешними ресурсами.
- **Преимущества**:
  - Улучшает производительность за счет создания объекта только один раз.
  - Позволяет создавать глобальные сервисы, доступные для всех компонентов приложения.
- **Пример**:
  ```csharp
  public class MySingletonService : IMyService
  {
      public void PerformAction()
      {
          // Логика сервиса
      }
  }

  // Регистрация сервиса
  services.AddSingleton<IMyService, MySingletonService>();
  ```

  Все запросы к `IMyService` будут получать один и тот же экземпляр `MySingletonService` на протяжении всего жизненного цикла приложения.

---

### Пример использования разных типов регистрации DI

Предположим, у нас есть три разных сервиса: `TransientService`, `ScopedService`, и `SingletonService`. Мы зарегистрируем их в DI-контейнере и используем в контроллере:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddTransient<ITransientService, TransientService>();
        services.AddScoped<IScopedService, ScopedService>();
        services.AddSingleton<ISingletonService, SingletonService>();
        services.AddControllers();
    }
}
```

Теперь в контроллере мы можем инжектировать эти сервисы и наблюдать различия:

```csharp
public class MyController : ControllerBase
{
    private readonly ITransientService _transientService;
    private readonly IScopedService _scopedService;
    private readonly ISingletonService _singletonService;

    public MyController(ITransientService transientService, IScopedService scopedService, ISingletonService singletonService)
    {
        _transientService = transientService;
        _scopedService = scopedService;
        _singletonService = singletonService;
    }

    [HttpGet]
    public IActionResult Get()
    {
        // Здесь вы можете вызывать методы сервисов
        _transientService.PerformAction();
        _scopedService.PerformAction();
        _singletonService.PerformAction();

        return Ok("DI services executed successfully");
    }
}
```

### Итоговое сравнение

| Тип регистрации | Время жизни                       | Применимость                                |
|-----------------|-----------------------------------|---------------------------------------------|
| **Transient**   | Каждый запрос — новый экземпляр   | Легковесные объекты, не сохраняющие состояние |
| **Scoped**      | Один экземпляр на HTTP-запрос     | Данные и операции, связанные с текущим запросом |
| **Singleton**   | Один экземпляр на время работы приложения | Кэширование, глобальные настройки приложения   |

Выбор между `AddTransient`, `AddScoped` и `AddSingleton` зависит от требований приложения к производительности, состоянию объектов и частоте их использования.
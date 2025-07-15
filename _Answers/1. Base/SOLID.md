> [!important]
> [[Solid(site)]]
## Определение

**SOLID** - это аббревиатура пяти основных принципов объектно-ориентированного программирования и проектирования, сформулированных Робертом Мартином (Uncle Bob). Эти принципы помогают создавать поддерживаемый, расширяемый и гибкий код.

## S - Single Responsibility Principle (Принцип единственной ответственности)

### Определение

Каждый класс должен иметь только одну причину для изменения, то есть только одну ответственность.

### Неправильно:

```csharp
// Класс нарушает SRP - у него несколько ответственностей
public class UserManager
{
    public void CreateUser(User user)
    {
        // Валидация
        if (string.IsNullOrEmpty(user.Email))
            throw new ArgumentException("Email is required");
        
        // Сохранение в БД
        using var connection = new SqlConnection("connectionString");
        connection.Open();
        var command = new SqlCommand("INSERT INTO Users...", connection);
        command.ExecuteNonQuery();
        
        // Отправка email
        var smtpClient = new SmtpClient("smtp.gmail.com");
        smtpClient.Send(new MailMessage("from@test.com", user.Email, "Welcome", "Welcome message"));
        
        // Логирование
        File.AppendAllText("log.txt", $"User {user.Email} created at {DateTime.Now}");
    }
}
```

### Правильно:

```csharp
// Каждый класс имеет одну ответственность
public class UserValidator
{
    public void Validate(User user)
    {
        if (string.IsNullOrEmpty(user.Email))
            throw new ArgumentException("Email is required");
    }
}

public class UserRepository
{
    public void Save(User user)
    {
        using var connection = new SqlConnection("connectionString");
        connection.Open();
        var command = new SqlCommand("INSERT INTO Users...", connection);
        command.ExecuteNonQuery();
    }
}

public class EmailService
{
    public void SendWelcomeEmail(User user)
    {
        var smtpClient = new SmtpClient("smtp.gmail.com");
        smtpClient.Send(new MailMessage("from@test.com", user.Email, "Welcome", "Welcome message"));
    }
}

public class Logger
{
    public void Log(string message)
    {
        File.AppendAllText("log.txt", $"{message} at {DateTime.Now}");
    }
}

public class UserService
{
    private readonly UserValidator _validator;
    private readonly UserRepository _repository;
    private readonly EmailService _emailService;
    private readonly Logger _logger;
    
    public UserService(UserValidator validator, UserRepository repository, 
                      EmailService emailService, Logger logger)
    {
        _validator = validator;
        _repository = repository;
        _emailService = emailService;
        _logger = logger;
    }
    
    public void CreateUser(User user)
    {
        _validator.Validate(user);
        _repository.Save(user);
        _emailService.SendWelcomeEmail(user);
        _logger.Log($"User {user.Email} created");
    }
}
```

## O - Open/Closed Principle (Принцип открытости/закрытости)

### Определение

Программные сущности должны быть открыты для расширения, но закрыты для модификации.

### Неправильно:

```csharp
public class DiscountCalculator
{
    public decimal CalculateDiscount(Customer customer, decimal amount)
    {
        if (customer.Type == CustomerType.Regular)
            return amount * 0.1m;
        else if (customer.Type == CustomerType.Premium)
            return amount * 0.2m;
        else if (customer.Type == CustomerType.VIP)
            return amount * 0.3m;
        
        return 0;
    }
}
```

### Правильно:

```csharp
public interface IDiscountStrategy
{
    decimal CalculateDiscount(decimal amount);
}

public class RegularCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount) => amount * 0.1m;
}

public class PremiumCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount) => amount * 0.2m;
}

public class VIPCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount) => amount * 0.3m;
}

public class DiscountCalculator
{
    private readonly Dictionary<CustomerType, IDiscountStrategy> _strategies;
    
    public DiscountCalculator()
    {
        _strategies = new Dictionary<CustomerType, IDiscountStrategy>
        {
            { CustomerType.Regular, new RegularCustomerDiscount() },
            { CustomerType.Premium, new PremiumCustomerDiscount() },
            { CustomerType.VIP, new VIPCustomerDiscount() }
        };
    }
    
    public decimal CalculateDiscount(Customer customer, decimal amount)
    {
        if (_strategies.TryGetValue(customer.Type, out var strategy))
            return strategy.CalculateDiscount(amount);
        
        return 0;
    }
}
```

## L - Liskov Substitution Principle (Принцип подстановки Лисков)

### Определение

Объекты суперкласса должны быть заменяемыми объектами подклассов без изменения корректности программы.

### Неправильно:

```csharp
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Bird is flying");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotSupportedException("Penguins can't fly"); // Нарушение LSP
    }
}

// Использование
void MakeBirdFly(Bird bird)
{
    bird.Fly(); // Может выбросить исключение для пингвина
}
```

### Правильно:

```csharp
public abstract class Bird
{
    public abstract void Move();
}

public interface IFlyable
{
    void Fly();
}

public class Sparrow : Bird, IFlyable
{
    public override void Move() => Fly();
    public void Fly() => Console.WriteLine("Sparrow is flying");
}

public class Penguin : Bird
{
    public override void Move() => Swim();
    public void Swim() => Console.WriteLine("Penguin is swimming");
}

// Использование
void MakeBirdMove(Bird bird)
{
    bird.Move(); // Всегда работает корректно
}

void MakeFlyableFly(IFlyable flyable)
{
    flyable.Fly(); // Работает только с летающими объектами
}
```

## I - Interface Segregation Principle (Принцип разделения интерфейса)

### Определение

Клиенты не должны зависеть от интерфейсов, которые они не используют.

### Неправильно:

```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Human : IWorker
{
    public void Work() => Console.WriteLine("Human working");
    public void Eat() => Console.WriteLine("Human eating");
    public void Sleep() => Console.WriteLine("Human sleeping");
}

public class Robot : IWorker
{
    public void Work() => Console.WriteLine("Robot working");
    public void Eat() => throw new NotImplementedException(); // Роботы не едят
    public void Sleep() => throw new NotImplementedException(); // Роботы не спят
}
```

### Правильно:

```csharp
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work() => Console.WriteLine("Human working");
    public void Eat() => Console.WriteLine("Human eating");
    public void Sleep() => Console.WriteLine("Human sleeping");
}

public class Robot : IWorkable
{
    public void Work() => Console.WriteLine("Robot working");
}

// Использование
void ManageWorkable(IWorkable worker)
{
    worker.Work(); // Работает с любым работником
}

void FeedCreature(IEatable creature)
{
    creature.Eat(); // Работает только с теми, кто может есть
}
```

## D - Dependency Inversion Principle (Принцип инверсии зависимостей)

### Определение

1. Модули высокого уровня не должны зависеть от модулей низкого уровня. Оба должны зависеть от абстракций.
2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

### Неправильно:

```csharp
public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        // Отправка email
        Console.WriteLine($"Sending email to {to}");
    }
}

public class OrderService
{
    private readonly EmailService _emailService; // Прямая зависимость
    
    public OrderService()
    {
        _emailService = new EmailService(); // Создание зависимости
    }
    
    public void ProcessOrder(Order order)
    {
        // Обработка заказа
        _emailService.SendEmail(order.CustomerEmail, "Order Confirmation", "Your order is confirmed");
    }
}
```

### Правильно:

```csharp
public interface INotificationService
{
    void SendNotification(string to, string subject, string body);
}

public class EmailService : INotificationService
{
    public void SendNotification(string to, string subject, string body)
    {
        Console.WriteLine($"Sending email to {to}");
    }
}

public class SMSService : INotificationService
{
    public void SendNotification(string to, string subject, string body)
    {
        Console.WriteLine($"Sending SMS to {to}");
    }
}

public class OrderService
{
    private readonly INotificationService _notificationService; // Зависимость от абстракции
    
    public OrderService(INotificationService notificationService) // Инъекция зависимости
    {
        _notificationService = notificationService;
    }
    
    public void ProcessOrder(Order order)
    {
        // Обработка заказа
        _notificationService.SendNotification(order.CustomerEmail, "Order Confirmation", "Your order is confirmed");
    }
}

// Конфигурация в DI-контейнере
public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<INotificationService, EmailService>();
    services.AddScoped<OrderService>();
}
```

## Комплексный пример применения SOLID

```csharp
// S - Single Responsibility
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task SaveAsync(User user);
}

public interface IEmailService
{
    Task SendAsync(string to, string subject, string body);
}

public interface IUserValidator
{
    ValidationResult Validate(User user);
}

// O - Open/Closed
public interface IUserCreationHandler
{
    Task HandleAsync(User user);
}

public class WelcomeEmailHandler : IUserCreationHandler
{
    private readonly IEmailService _emailService;
    
    public WelcomeEmailHandler(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public async Task HandleAsync(User user)
    {
        await _emailService.SendAsync(user.Email, "Welcome", "Welcome to our platform!");
    }
}

// L - Liskov Substitution
public abstract class UserNotification
{
    public abstract Task SendAsync(User user);
}

public class EmailNotification : UserNotification
{
    private readonly IEmailService _emailService;
    
    public EmailNotification(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public override async Task SendAsync(User user)
    {
        await _emailService.SendAsync(user.Email, "Notification", "You have a new notification");
    }
}

// I - Interface Segregation
public interface IReadOnlyUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<IEnumerable<User>> GetAllAsync();
}

public interface IWriteOnlyUserRepository
{
    Task SaveAsync(User user);
    Task DeleteAsync(int id);
}

// D - Dependency Inversion
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly IUserValidator _validator;
    private readonly IEnumerable<IUserCreationHandler> _creationHandlers;
    
    public UserService(
        IUserRepository userRepository,
        IUserValidator validator,
        IEnumerable<IUserCreationHandler> creationHandlers)
    {
        _userRepository = userRepository;
        _validator = validator;
        _creationHandlers = creationHandlers;
    }
    
    public async Task<Result> CreateUserAsync(User user)
    {
        var validationResult = _validator.Validate(user);
        if (!validationResult.IsValid)
            return Result.Failure(validationResult.Errors);
        
        await _userRepository.SaveAsync(user);
        
        // Обработка создания пользователя (O - открыт для расширения)
        foreach (var handler in _creationHandlers)
        {
            await handler.HandleAsync(user);
        }
        
        return Result.Success();
    }
}
```

## Преимущества применения SOLID

1. **Поддерживаемость**: Код легче читать, понимать и изменять
2. **Тестируемость**: Легче писать unit-тесты благодаря слабой связности
3. **Расширяемость**: Новый функционал добавляется без изменения существующего кода
4. **Повторное использование**: Компоненты можно использовать в разных контекстах
5. **Стабильность**: Изменения в одной части системы не влияют на другие

## Потенциальные проблемы

1. **Сложность**: Может усложнить простые задачи
2. **Абстракции**: Избыточные абстракции могут затруднить понимание
3. **Производительность**: Дополнительные уровни абстракции могут влиять на производительность

## Заключение

SOLID принципы являются фундаментальными для создания качественного объектно-ориентированного кода. Их правильное применение приводит к созданию гибких, поддерживаемых и расширяемых систем, что критически важно для enterprise-разработки на платформе .NET.
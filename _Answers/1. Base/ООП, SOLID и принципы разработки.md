## [1] Введение

Современная разработка программного обеспечения требует создания систем, которые легко поддерживать, масштабировать и развивать. Для этого важно использовать проверенные подходы и принципы проектирования. Объектно‑ориентированное программирование (ООП), SOLID и общие принципы разработки играют ключевую роль в достижении этой цели.

**Зачем это нужно?**

- **Улучшение структуры кода.** ООП и принципы проектирования помогают создавать логически организованный код, который легче понимать и расширять.
- **Гибкость.** Хорошо спроектированный код легче адаптируется к изменениям, что особенно важно в условиях динамичных требований.
- **Повышение надёжности.** Чёткие принципы и паттерны позволяют избежать множества ошибок и делают систему более устойчивой к сбоям.
- **Ускорение разработки.** Следование KISS и DRY упрощает процесс разработки и сокращает вероятность серьёзных ошибок.
- **Усиление масштабируемости.** SOLID и похожие принципы помогают создавать архитектуру, готовую к новым задачам и модулям.

## [2] ООП

**Объектно‑ориентированное программирование (ООП)** — это парадигма программирования, в которой данные и функции, работающие с ними, объединяются в единые структуры — объекты.

### Ключевые концепции:

1. **Абстракция** — скрытие деталей реализации и показ только необходимых элементов.
    
    ```csharp
    // Неправильно: все детали процесса приготовления кофе
    class CoffeeMachine {
      void HeatWater() { /* … */ }
      void GrindBeans() { /* … */ }
      void Brew()      { /* … */ }
    }
    // Правильно: только метод работы
    class CoffeeMachine {
      public void BrewCoffee() {
        Console.WriteLine("Кофе готовится...");
      }
    }
    ```
    
2. **Инкапсуляция** — объединение данных и методов в одном классе и скрытие ненужных деталей от внешнего мира.
    
    ```csharp
    class BankAccount {
      private decimal _balance;  // закрытое поле
    
      public void Deposit(decimal amount) {
        if (amount > 0) _balance += amount;
      }
      public void Withdraw(decimal amount) {
        if (amount > 0 && amount <= _balance) _balance -= amount;
      }
      public decimal GetBalance() => _balance;
    }
    ```
    
3. **Наследование** — позволяет создать класс‑наследник на основе существующего, расширив или изменив его поведение.
    
    ```csharp
    class Animal {
      public virtual void Move() => Console.WriteLine("Животное двигается");
    }
    class Dog : Animal {
      public override void Move() => Console.WriteLine("Собака бежит");
    }
    ```
    
4. **Полиморфизм** — позволяет объектам разных классов реагировать по‑разному на один и тот же вызов метода.
    
    ```csharp
    class Animal {
      public virtual void MakeSound() => Console.WriteLine("Животное издаёт звук");
    }
    class Dog : Animal {
      public override void MakeSound() => Console.WriteLine("Гав‑гав");
    }
    class Cat : Animal {
      public override void MakeSound() => Console.WriteLine("Мяу");
    }
    ```
    

## [3] SOLID

SOLID — набор пяти принципов объектно‑ориентированного проектирования, предложенных Робертом Мартином.

### S — Single Responsibility Principle (SRP)

Класс должен иметь только одну причину для изменения.

```csharp
class UserManager {
  public void AddUser(string name) { /* … */ }
}
class Logger {
  public void Log(string message) => Console.WriteLine($"Log: {message}");
}
```

### O — Open/Closed Principle (OCP)

Классы должны быть открыты для расширения, но закрыты для модификации.

```csharp
abstract class PaymentMethod {
  public abstract void Process(decimal amount);
}
class CreditCard : PaymentMethod {
  public override void Process(decimal amount) { /* … */ }
}
class PayPal : PaymentMethod {
  public override void Process(decimal amount) { /* … */ }
}
```

### L — Liskov Substitution Principle (LSP)

Наследник должен без проблем заменять базовый класс.

```csharp
class Bird {
  public virtual void Fly() { /* … */ }
}
class Sparrow : Bird { }
class NonFlyingBird : Bird {
  public override void Fly() => throw new NotSupportedException();
}
```

### I — Interface Segregation Principle (ISP)

Клиенты не должны зависеть от интерфейсов, которые они не используют.

```csharp
interface ICleaner { void Clean(); }
interface IBuilder { void Build(); }
```

### D — Dependency Inversion Principle (DIP)

Модули верхнего уровня не должны зависеть от модулей нижнего уровня. Оба должны зависеть от абстракций.

```csharp
interface ISender { void Send(string message); }
class EmailSender : ISender { public void Send(string msg) => /* … */; }
class NotificationService {
  private readonly ISender _sender;
  public NotificationService(ISender sender) => _sender = sender;
  public void Notify(string msg) => _sender.Send(msg);
}
```

## [4] Принципы разработки

- **KISS** (Keep It Short and Simple)
- **DRY** (Don’t Repeat Yourself)
- **YAGNI** (You Aren’t Gonna Need It)
- **Occam’s Razor**
- **BDUF** (Big Design Up Front)
- **APO** (Avoid Premature Optimization)

## [5] Заключение

ООП, SOLID и общие принципы разработки помогают создавать качественный, надёжный и масштабируемый код. Их применение снижает риск ошибок и упрощает развитие проектов.

**Рекомендации для дальнейшего чтения:**

- [ООП (Metanit)](https://metanit.com/common/langs/1.3.php)
- [SOLID (Habr)](https://habr.com/ru/companies/productivity_inside/articles/505430/)
- [Принципы разработки (TProger)](https://tproger.ru/articles/5-principov-chitaemogo-koda-kiss-yagni-dry-bduf-i-britva-okkama)
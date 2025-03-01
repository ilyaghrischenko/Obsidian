SOLID — это пять основных принципов объектно-ориентированного программирования, которые помогают создавать гибкие, поддерживаемые и масштабируемые системы.

## 1. **Single Responsibility Principle (SRP) — Принцип единственной ответственности**
Каждый класс должен иметь только одну причину для изменения, т.е. выполнять лишь одну задачу.

**Пример (нарушение принципа):**
```csharp
public class Report
{
    public void GenerateReport() { /* Генерация отчёта */ }
    public void SaveToFile() { /* Сохранение в файл */ }
}
```
**Исправленный вариант:**
```csharp
public class ReportGenerator
{
    public void GenerateReport() { /* Генерация отчёта */ }
}

public class FileSaver
{
    public void SaveToFile(string content) { /* Сохранение в файл */ }
}
```

---
## 2. **Open/Closed Principle (OCP) — Принцип открытости/закрытости**
Классы должны быть **открыты** для расширения, но **закрыты** для модификации.

**Неправильно:**
```csharp
public class PaymentProcessor
{
    public void ProcessPayment(string paymentType)
    {
        if (paymentType == "CreditCard") { /* обработка */ }
        else if (paymentType == "PayPal") { /* обработка */ }
    }
}
```
**Исправленный вариант:**
```csharp
public interface IPaymentMethod
{
    void ProcessPayment();
}

public class CreditCardPayment : IPaymentMethod
{
    public void ProcessPayment() { /* обработка */ }
}

public class PayPalPayment : IPaymentMethod
{
    public void ProcessPayment() { /* обработка */ }
}
```

---
## 3. **Liskov Substitution Principle (LSP) — Принцип подстановки Барбары Лисков**
Дочерние классы должны без изменений подставляться вместо родительских.

**Нарушение принципа:**
```csharp
public class Bird
{
    public virtual void Fly() { }
}

public class Penguin : Bird
{
    public override void Fly() { throw new Exception("Пингвины не летают!"); }
}
```
**Правильный вариант:**
```csharp
public abstract class Bird { }

public class FlyingBird : Bird
{
    public void Fly() { }
}

public class Penguin : Bird { }
```

---
## 4. **Interface Segregation Principle (ISP) — Принцип разделения интерфейсов**
Не стоит заставлять классы реализовывать интерфейсы, которые они не используют.

**Неправильно:**
```csharp
public interface IWorker
{
    void Work();
    void Eat();
}

public class Robot : IWorker
{
    public void Work() { }
    public void Eat() { throw new NotImplementedException(); }
}
```
**Исправленный вариант:**
```csharp
public interface IWorkable { void Work(); }
public interface IEatable { void Eat(); }

public class Robot : IWorkable
{
    public void Work() { }
}
```

---
## 5. **Dependency Inversion Principle (DIP) — Принцип инверсии зависимостей**
Модули верхнего уровня не должны зависеть от модулей нижнего уровня. Оба должны зависеть от абстракций.

**Нарушение принципа:**
```csharp
public class EmailService
{
    public void SendEmail() { /* Отправка email */ }
}

public class Notification
{
    private EmailService emailService = new EmailService();
    public void Send() { emailService.SendEmail(); }
}
```
**Правильный вариант:**
```csharp
public interface IMessageService
{
    void SendMessage();
}

public class EmailService : IMessageService
{
    public void SendMessage() { /* Отправка email */ }
}

public class Notification
{
    private readonly IMessageService _messageService;
    
    public Notification(IMessageService messageService)
    {
        _messageService = messageService;
    }
    
    public void Send() { _messageService.SendMessage(); }
}
```

---
## Итог
Принципы SOLID помогают разрабатывать гибкие и масштабируемые системы, упрощают поддержку кода и уменьшают вероятность появления ошибок. Использование этих принципов делает код чище, понятнее и более модульным.

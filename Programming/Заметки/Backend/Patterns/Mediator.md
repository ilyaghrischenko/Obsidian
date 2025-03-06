# MediatR и паттерн Mediator в `C#`#

## 1. Паттерн Mediator

### Что такое Mediator?

**Mediator** (Посредник) — это поведенческий паттерн проектирования, который уменьшает связанность компонентов системы, позволяя им взаимодействовать друг с другом через централизованный объект (посредник). Это улучшает масштабируемость и поддержку кода.

### Преимущества Mediator:

- **Снижение связанности** между объектами.
- **Упрощение поддержки** кода.
- **Гибкость и расширяемость** системы.
- **Централизация логики взаимодействия** между объектами.

### Пример без Mediator (сильная связанность):

```csharp
public class User
{
    public string Name { get; set; }
    public Logger Logger { get; set; }
    
    public void ChangeName(string name)
    {
        Name = name;
        Logger.Log($"User changed name to: {Name}");
    }
}

public class Logger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}

var user = new User { Name = "Illia", Logger = new Logger() };
user.ChangeName("New Name");
```

### Пример с Mediator (слабая связанность):

```csharp
public class User
{
    public string Name { get; set; }
    private readonly IMediator _mediator;

    public User(IMediator mediator, string name)
    {
        _mediator = mediator;
        Name = name;
    }

    public async Task ChangeNameAsync(string name)
    {
        Name = name;
        await _mediator.Publish(new UserUpdated(this));
    }
}

public class UserUpdated : INotification
{
    public User User { get; }
    public UserUpdated(User user) => User = user;
}

public class Logger : INotificationHandler<UserUpdated>
{
    public Task Handle(UserUpdated notification, CancellationToken cancellationToken)
    {
        Console.WriteLine($"User changed name to: {notification.User.Name}");
        return Task.CompletedTask;
    }
}
```

---

## 2. Библиотека MediatR

**MediatR** — это библиотека, реализующая паттерн **Mediator** в C#. Она используется для упрощения коммуникации между объектами через централизованный диспетчер сообщений.

### Основные концепции MediatR:

- **Запросы (Requests) и обработчики (Handlers)** — используются для **CQRS** (разделение команд и запросов).
- **Уведомления (Notifications) и обработчики (Handlers)** — используются для событийной модели.
- **Pipeline Behaviors** — позволяют внедрять логику (например, логирование, кэширование) в конвейер запросов.

### Установка MediatR

```sh
Install-Package MediatR
Install-Package Microsoft.Extensions.DependencyInjection
```

### Пример использования MediatR

#### Определяем модель пользователя

```csharp
using MediatR;

public class User
{
    public required string Name { get; set; }
    public required ushort Age { get; set; }
    
    public required IMediator Mediator { get; set; }

    private async Task InvokeAsync()
    {
        await Mediator.Publish(new UserUpdated { UpdatedUser = this });
    }

    public async Task ChangeName(string name)
    {
        Name = name;
        await InvokeAsync();
    }

    public async Task ChangeAge(ushort age)
    {
        Age = age;
        await InvokeAsync();
    }

    public override string ToString() => $"Name: {Name}, Age: {Age}";
}
```

#### Определяем событие обновления пользователя

```csharp
using MediatR;

public class UserUpdated : INotification
{
    public required User UpdatedUser { get; init; }
}
```

#### Логгер как обработчик события

```csharp
using MediatR;

public class Logger : INotificationHandler<UserUpdated>
{
    public Task Handle(UserUpdated notification, CancellationToken cancellationToken)
    {
        Console.WriteLine("Data changed!!\n\n");
        Console.WriteLine($"""
            Name: {notification.UpdatedUser.Name}
            Age: {notification.UpdatedUser.Age}
        """);
        return Task.CompletedTask;
    }
}
```

#### Регистрация в DI-контейнере и запуск

```csharp
using Mediator;
using MediatR;
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();
services.AddMediatR(cfg => cfg.RegisterServicesFromAssemblyContaining<UserUpdated>());
services.AddTransient<INotificationHandler<UserUpdated>, Logger>();

var provider = services.BuildServiceProvider();
var mediator = provider.GetRequiredService<IMediator>();

User user = new() { Name = "Illia", Age = 19, Mediator = mediator };

Console.WriteLine($"{user}\n\n");

await user.ChangeName("NEW NAME");

Console.ReadLine();
```

---

## 3. Итог

### Когда использовать Mediator?

✅ В сложных системах с высокой связанностью компонентов. ✅ Когда требуется гибкость в расширении системы. ✅ Для упрощения логики взаимодействий между объектами.

### Когда **не стоит** использовать Mediator?

❌ В небольших и простых проектах. ❌ Когда производительность критична (Mediator добавляет небольшую накладную стоимость).

**MediatR** — мощный инструмент для реализации паттерна **Mediator** в C#. Он позволяет уменьшить связанность компонентов, упростить поддержку кода и сделать архитектуру более гибкой.
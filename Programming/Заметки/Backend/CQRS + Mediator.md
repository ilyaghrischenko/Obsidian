tags: [#backend, #cqrs, #mediator]

## Что такое [[CQRS]]?

**CQRS (Command Query Responsibility Segregation)** — это архитектурный паттерн, разделяющий операции **чтения** (Query) и **изменения состояния** (Command). Это позволяет:

- Улучшить производительность, масштабируемость и безопасность системы.
- Избежать конфликтов при конкурентных изменениях данных.
- Гибко разделять ответственность между разными компонентами.

## Связь CQRS и [[Mediator]]

Библиотека **MediatR** отлично подходит для реализации CQRS, так как она помогает:

- Отделить обработку команд и запросов.
- Уменьшить связанность кода.
- Упростить тестирование и поддержку.

---

## Реализация CQRS + MediatR на C#

### 1. Определение команды (Command)

```csharp
using MediatR;

public record UpdateUserCommand(string Name, ushort Age) : IRequest;
```

### 2. Обработчик команды (Command Handler)

```csharp
public class UpdateUserCommandHandler : IRequestHandler<UpdateUserCommand>
{
    public Task Handle(UpdateUserCommand request, CancellationToken cancellationToken)
    {
        Console.WriteLine($"User updated: {request.Name}, {request.Age}");
        return Task.CompletedTask;
    }
}
```

### 3. Определение запроса (Query)

```csharp
public record GetUserQuery(string Name) : IRequest<User>;
```

### 4. Обработчик запроса (Query Handler)

```csharp
public class GetUserQueryHandler : IRequestHandler<GetUserQuery, User>
{
    public Task<User> Handle(GetUserQuery request, CancellationToken cancellationToken)
    {
        var user = new User { Name = request.Name, Age = 25 };
        return Task.FromResult(user);
    }
}
```

### 5. Конфигурация и запуск

```csharp
using Microsoft.Extensions.DependencyInjection;
using MediatR;

var services = new ServiceCollection();
services.AddMediatR(cfg => cfg.RegisterServicesFromAssemblyContaining<UpdateUserCommand>());
var provider = services.BuildServiceProvider();
var mediator = provider.GetRequiredService<IMediator>();

await mediator.Send(new UpdateUserCommand("Illia", 19));
var user = await mediator.Send(new GetUserQuery("Illia"));
Console.WriteLine($"User fetched: {user.Name}, {user.Age}");
```

---

## Заключение

Использование **CQRS + MediatR** помогает структурировать код и делает его более модульным. Этот подход особенно полезен в крупных системах, где нужна четкая организация команд и запросов. 🚀
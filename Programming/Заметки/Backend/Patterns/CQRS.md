tags: [#backend, #patterns, #cqrs]

## Что такое CQRS?

**CQRS (Command Query Responsibility Segregation)** — это архитектурный паттерн, разделяющий операции **чтения** (Query) и **изменения состояния** (Command). Этот подход позволяет строить более масштабируемые и удобные для сопровождения системы.

## Основные принципы CQRS

1. **Разделение ответственности**: команды (Commands) изменяют состояние системы, а запросы (Queries) только читают данные.
2. **Изоляция нагрузки**: чтение и запись могут использовать разные базы данных или механизмы хранения.
3. **Гибкость и масштабируемость**: позволяет оптимизировать чтение и запись независимо друг от друга.
4. **Поддержка событийной модели**: CQRS часто используется вместе с Event Sourcing для более четкой истории изменений данных.

## Пример традиционного подхода (до CQRS)

```csharp
public class UserService
{
    private readonly DatabaseContext _context;
    
    public UserService(DatabaseContext context) => _context = context;

    public User GetUser(int id) => _context.Users.Find(id);

    public void UpdateUser(User user)
    {
        var existingUser = _context.Users.Find(user.Id);
        existingUser.Name = user.Name;
        _context.SaveChanges();
    }
}
```

### Проблемы

- Один класс выполняет как **чтение**, так и **изменение** данных.
- Трудно масштабировать, так как нагрузка на чтение и запись растет одновременно.
- Сложно внедрять кеширование и оптимизацию.

## CQRS-решение

### Разделение на команды и запросы:

```csharp
// Запрос (Query) — только чтение
public class GetUserQuery : IRequest<User>
{
    public int Id { get; }
    public GetUserQuery(int id) => Id = id;
}

public class GetUserHandler : IRequestHandler<GetUserQuery, User>
{
    private readonly DatabaseContext _context;
    public GetUserHandler(DatabaseContext context) => _context = context;
    public Task<User> Handle(GetUserQuery request, CancellationToken cancellationToken)
        => Task.FromResult(_context.Users.Find(request.Id));
}
```

```csharp
// Команда (Command) — изменяет состояние
public class UpdateUserCommand : IRequest
{
    public int Id { get; }
    public string Name { get; }
    public UpdateUserCommand(int id, string name)
    {
        Id = id;
        Name = name;
    }
}

public class UpdateUserHandler : IRequestHandler<UpdateUserCommand>
{
    private readonly DatabaseContext _context;
    public UpdateUserHandler(DatabaseContext context) => _context = context;
    public Task Handle(UpdateUserCommand request, CancellationToken cancellationToken)
    {
        var user = _context.Users.Find(request.Id);
        user.Name = request.Name;
        _context.SaveChanges();
        return Task.CompletedTask;
    }
}
```

## Преимущества CQRS

✅ Улучшенная производительность и масштабируемость. ✅ Лучшая поддержка кеширования. ✅ Упрощенная поддержка событий и истории изменений (Event Sourcing). ✅ Облегченная безопасность за счет разделения операций.

## Когда использовать CQRS?

✅ В системах с высокой нагрузкой (где много чтения и записи). ✅ В сложных доменных моделях (где логика команд и запросов сильно различается). ✅ В распределенных системах, где важно разделять операции.

## Когда НЕ стоит использовать CQRS?

❌ В простых приложениях, где разделение команд и запросов избыточно. ❌ В проектах с малым количеством операций над данными.

## Заключение

CQRS — мощный инструмент для создания масштабируемых и гибких приложений. Однако он требует дополнительных усилий в разработке и должен использоваться там, где это действительно необходимо. 🚀
## Что такое Vertical Sliced Architecture

Vertical Sliced Architecture (VSA) — это подход к организации кода, в котором приложение делится не на слои (как в Clean Architecture), а на вертикальные куски функционала, чаще всего определяемые по контроллеру или по одному рейкуесту.

## Особенности VSA

- Код делится по контекстам/фичам (features), а не по слоям (Controller, Service, Repository).
    
- Каждая фича является самодостаточной.
    
- Хорошо подходит для CQRS (разделение Command/Query).
    

## Структура проекта при VSA

```
ProjectRoot/
|-- Features/
|   |-- Products/
|   |   |-- Create/
|   |   |   |-- CreateProductCommand.cs
|   |   |   |-- CreateProductHandler.cs
|   |   |   |-- CreateProductValidator.cs
|   |   |-- Get/
|   |   |   |-- GetProductsQuery.cs
|   |   |   |-- GetProductsHandler.cs
|-- Infrastructure/
|-- Program.cs
|-- Startup.cs (if used)
```

## Каждая Feature:

### Command:

```csharp
public record CreateProductCommand(string Name, decimal Price);
```

### Handler:

```csharp
public class CreateProductHandler : IRequestHandler<CreateProductCommand>
{
    private readonly AppDbContext _context;

    public CreateProductHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        var product = new Product { Name = request.Name, Price = request.Price };
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
    }
}
```

### Validator (optional):

```csharp
public class CreateProductValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name).NotEmpty();
        RuleFor(x => x.Price).GreaterThan(0);
    }
}
```

## Роутинг

Обычно используют Minimal API или Mediator:

```csharp
app.MapPost("/products", async (IMediator mediator, CreateProductCommand command) =>
{
    await mediator.Send(command);
    return Results.Ok();
});
```

## Преимущества

- Легко организовывать и создавать модули
    
- Хорошо читается код
    
- Каждая фича локализована, легче тестировать
    

## Недостатки

- Может выглядеть как бесструктурный код без опыта
    
- Труднее следить за повтором кода в крупных проектах
    

## С чем чаще всего сочетают VSA

- MediatR (Command/Query Handlers)
    
- FluentValidation
    
- Entity Framework Core
    
- Minimal APIs или MVC Controllers
    

## Вывод

Vertical Sliced Architecture — это удобный, масштабируемый подход для больших и средних проектов, позволяющий делить логику по задачам/фичам, а не по типам слоев.
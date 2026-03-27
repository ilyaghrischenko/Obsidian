tags: [#backend, #libraries, #scrutor]

## Что такое Scrutor?

**Scrutor** - это библиотека для расширения стандартного контейнера Dependency Injection в `.NET`. Она позволяет удобно регистрировать сервисы через сканирование сборок и применять `decorator`-подход без ручного перечисления каждой зависимости.

На практике `Scrutor` особенно полезен в проектах на [[ASP.NET Core MVC]] и [[Minimal API]], где количество сервисов быстро растет и ручная регистрация через `builder.Services.AddScoped<...>()` начинает раздувать `Program.cs`.

## Зачем использовать?

- Уменьшает количество ручной регистрации сервисов.
- Позволяет автоматически находить и подключать классы по соглашениям.
- Упрощает поддержку больших решений с большим количеством сервисов и обработчиков.
- Поддерживает `decorator`-подход для логирования, кеширования и оберток.
- Хорошо сочетается с [[DI injection types]] и вынесением конфигурации в [[WebApplicationBuilderExtensions]].

## Как это работает?

`Scrutor` добавляет удобные расширения поверх стандартного DI-контейнера:

- `Scan(...)` для автоматической регистрации типов;
- `Decorate(...)` для оборачивания сервиса дополнительной логикой.

### ✅ Базовый пример `Scan`

```csharp
builder.Services.Scan(scan => scan
    .FromAssemblyOf<IUserService>()
    .AddClasses(classes => classes.AssignableTo<IUserService>())
    .AsImplementedInterfaces()
    .WithScopedLifetime());
```

В этом примере библиотека находит подходящие классы в сборке и регистрирует их как реализации интерфейсов со `scoped` lifetime.

### ⚙️ Пример по namespace или суффиксу

```csharp
builder.Services.Scan(scan => scan
    .FromAssemblies(typeof(ApplicationAssemblyMarker).Assembly)
    .AddClasses(classes => classes.Where(type => type.Name.EndsWith("Service")))
    .AsImplementedInterfaces()
    .WithScopedLifetime());
```

Так удобно регистрировать все сервисы, которые следуют единому naming convention.

### ✅ Пример с extension method

Если проект уже использует подход через [[WebApplicationBuilderExtensions]], регистрацию можно вынести отдельно:

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.Scan(scan => scan
            .FromAssemblies(typeof(ApplicationAssemblyMarker).Assembly)
            .AddClasses(classes => classes.InNamespaces("MyApp.Application.Services"))
            .AsImplementedInterfaces()
            .WithScopedLifetime());

        return services;
    }
}
```

Подключение:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddApplicationServices();
```

### ✅ Пример `Decorate`

`Scrutor` умеет оборачивать уже зарегистрированный сервис дополнительным слоем логики:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.Decorate<IOrderService, LoggingOrderService>();
```

Пример декоратора:

```csharp
public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger<LoggingOrderService> _logger;

    public LoggingOrderService(IOrderService inner, ILogger<LoggingOrderService> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task CreateAsync(Order order, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Creating order {OrderId}", order.Id);
        await _inner.CreateAsync(order, cancellationToken);
    }
}
```

Такой подход хорошо вписывается в принципы [[SOLID]], потому что позволяет добавлять поведение без изменения основного сервиса.

### ⚠️ Что важно помнить

- `Scrutor` не заменяет стандартный DI-контейнер, а расширяет его.
- Автоматическая регистрация удобна, но требует понятных соглашений по именам и структуре проекта.
- При чрезмерно широком `Scan` можно случайно зарегистрировать лишние типы.
- Для сложной логики регистрации лучше явно ограничивать сборки, namespace и lifetime.

## Заключение

**Scrutor** полезен, когда в проекте становится слишком много DI-регистрации вручную. Он помогает сделать конфигурацию чище, сократить копипаст и удобно внедрять `decorator`-подход поверх стандартного контейнера `.NET`.

## Что такое Middleware?
Middleware — это промежуточные компоненты в конвейере обработки HTTP-запросов в ASP.NET Core. Они выполняют обработку запросов и ответов, могут изменять их или передавать дальше по цепочке.

## Основные функции Middleware:
- Логирование запросов
- Обработка ошибок
- Аутентификация и авторизация
- Кэширование
- Манипуляция HTTP-запросами и ответами

## Подключение Middleware
Middleware добавляются в конвейер обработки ([[ASP.NET Core Pipeline]]) в файле `Program.cs`.

Пример:
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseMiddleware<CustomMiddleware>();

app.Run(async (context) =>
{
    await context.Response.WriteAsync("Hello, World!");
});

app.Run();
```

## Создание собственного Middleware

Создадим Middleware для логирования запросов (можно написать с использованием [[Serilog]]).

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");
        await _next(context);
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    }
}
```

### Добавление Middleware в конвейер:
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseMiddleware<LoggingMiddleware>();

app.Run(async (context) =>
{
    await context.Response.WriteAsync("Middleware example");
});

app.Run();
```

## Использование встроенных Middleware
ASP.NET Core предоставляет ряд встроенных Middleware:

### 1. **`UseRouting`** — маршрутизация
```csharp
app.UseRouting();
```

### 2. **`UseAuthentication`** — аутентификация
```csharp
app.UseAuthentication();
```

### 3. **`UseAuthorization`** — авторизация
```csharp
app.UseAuthorization();
```

### 4. **`UseExceptionHandler`** — обработка ошибок
```csharp
app.UseExceptionHandler("/error");
```

### 5. **`UseCors`** — настройка CORS (Cross-Origin Resource Sharing)
```csharp
app.UseCors(policy => policy.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());
```

## Итог
Middleware — это мощный механизм обработки HTTP-запросов в ASP.NET Core. Они позволяют удобно настраивать обработку запросов, добавлять кастомную логику и использовать встроенные механизмы ASP.NET Core для маршрутизации, аутентификации, логирования и других задач.

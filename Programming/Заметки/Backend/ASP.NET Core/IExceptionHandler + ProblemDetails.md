## **Что это такое**

- **IExceptionHandler** — интерфейс в ASP.NET Core 7+, позволяющий централизованно обрабатывать исключения.
    
- **ProblemDetails** — стандартный формат ответа об ошибке по [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807), который описывает ошибки в HTTP API.
    


Комбинация этих подходов позволяет:

- Иметь единый стиль ошибок в API.
    
- Возвращать детализированные и структурированные ответы.
    
- Отделить обработку исключений от бизнес-логики.
    

## **Зачем нужно**


✅ Преимущества:

- Единый формат ошибок для клиента.
    
- Минимум дублирования кода (не нужно в каждом контроллере try-catch).
    
- Легко расширять и настраивать.
    

❌ Без этого:

- В разных местах могут быть разные форматы ошибок.
    
- Сложно централизованно логировать.
    
- Код контроллеров перегружается try-catch блоками.
    

## **Использование**


### **1. Реализация ExceptionHandler**

``` csharp
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;

namespace Wolfix.Host.ExceptionHandlers;

public sealed class ExceptionHandler(IProblemDetailsService problemDetailsService) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
    {
        (int statusCode, string title) = exception switch
        {
            OperationCanceledException => (499, "ClientClosedRequest"),
            _ => (500, "InternalServerError")
        };

        ProblemDetails problemDetails = new()
        {
            Title = title,
            Status = statusCode,
            Detail = exception.Message,
            Instance = httpContext.Request.Path
        };

        httpContext.Response.StatusCode = statusCode;

        return await problemDetailsService.TryWriteAsync(new ProblemDetailsContext
        {
            HttpContext = httpContext,
            ProblemDetails = problemDetails,
            Exception = exception,
        });
    }
}
```

### **2. Регистрация в** 

### **Program.cs**

``` csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddExceptionHandler<ExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
app.MapControllers();

app.Run();
```

### **3. Расширение логики под разные исключения**

  

Можно добавлять собственные типы исключений и маппить их на нужный статус:

``` csharp
(int statusCode, string title) = exception switch
{
    UnauthorizedAccessException => (401, "Unauthorized"),
    KeyNotFoundException => (404, "NotFound"),
    ArgumentException => (400, "BadRequest"),
    OperationCanceledException => (499, "ClientClosedRequest"),
    _ => (500, "InternalServerError")
};
```

## **Пример ответа API**

``` csharp
{
  "type": "about:blank",
  "title": "NotFound",
  "status": 404,
  "detail": "Пользователь с таким ID не найден",
  "instance": "/api/users/123"
}
```

## **Важные моменты**

- ProblemDetails — стандарт, который понимают многие клиенты.
    
- IExceptionHandler позволяет отделить **техническую обработку** ошибок от бизнес-логики.
    
- Хорошая практика — логировать исключения (через ILogger).
    
- Подходит как для REST API, так и для [[Minimal API]].
    

  

## **Вывод**

- IExceptionHandler + ProblemDetails = современный и правильный подход для централизованной обработки ошибок в ASP.NET Core.
    
- Делает API предсказуемым и удобным для клиентов.
    
- Упрощает сопровождение и масштабирование кода.
## **📌 Что такое Health Checks?**

  

Health Checks — это механизм ASP.NET Core, позволяющий отслеживать состояние приложения и его зависимостей, таких как база данных, кэш, внешние сервисы и т.д.

  

## **⚙️ Установка**

  

Если вы используете ASP.NET Core 3.0 и выше, ничего дополнительно устанавливать не нужно. Для расширенных проверок (например, SQL Server, Redis и т.д.) установите соответствующие пакеты:

``` csharp
dotnet add package AspNetCore.HealthChecks.SqlServer
dotnet add package AspNetCore.HealthChecks.Redis
```

## **✅ Базовая настройка**


### **Program.cs (для .NET 6/7/8)**

``` csharp
var builder = WebApplication.CreateBuilder(args);

// Добавляем Health Checks
builder.Services.AddHealthChecks();

var app = builder.Build();

// Маршрут Health Check
app.MapHealthChecks("/health");

app.Run();
```

## **🔍 Расширенные проверки**

``` csharp
builder.Services.AddHealthChecks()
    .AddSqlServer(
        connectionString: builder.Configuration.GetConnectionString("DefaultConnection"),
        name: "sql",
        failureStatus: HealthStatus.Unhealthy)
    .AddRedis(
        redisConnectionString: builder.Configuration.GetConnectionString("Redis"),
        name: "redis",
        failureStatus: HealthStatus.Degraded);
```

## **🌐 Вывод результатов в JSON**

  

Чтобы получить структурированный JSON-ответ, используйте HealthCheckOptions:

``` csharp
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description
            }),
            totalDuration = report.TotalDuration
        });

        await context.Response.WriteAsync(result);
    }
});
```

## **📡 Примеры использования**

- ✅ Проверка доступности базы данных
    
- 🔐 Проверка подключения к Redis/Elasticsearch
    
- 🌍 Проверка внешнего HTTP API
    
- 💾 Проверка свободного места на диске
    

  

## **🛡️ Использование с Kubernetes**

  

Health Checks можно использовать для:

- **Liveness**-проб (/health/live) — жив ли процесс
    
- **Readiness**-проб (/health/ready) — готово ли приложение к приему трафика

``` csharp
builder.Services.AddHealthChecks()
    .AddCheck<LivenessCheck>("self");

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = check => check.Name == "self"
});
```

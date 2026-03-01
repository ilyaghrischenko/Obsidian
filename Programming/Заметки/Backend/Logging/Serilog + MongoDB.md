
Использование MongoDB в качестве хранилища логов позволяет хранить данные в виде документов (BSON), что значительно упрощает поиск и фильтрацию по сложным свойствам по сравнению с обычными текстовыми файлами.

## 1. Необходимые пакеты (NuGet)

Для реализации базового функционала и интеграции с Web API установи следующие зависимости:

``` shell
# Базовая интеграция Serilog в ASP.NET Core
dotnet add package Serilog.AspNetCore
# Синк для записи логов в MongoDB
dotnet add package Serilog.Sinks.MongoDB
# Чтение конфигурации из appsettings.json
dotnet add package Serilog.Settings.Configuration
# Загрузка переменных из .env (рекомендуется для секретов)
dotnet add package DotNetEnv
```

## 2. Безопасная конфигурация (.env)

Не храни строку подключения в `appsettings.json`. Используй переменные окружения.

**Файл `.env`:**

``` env
MONGODB_LOGGING_DATABASE_URL=mongodb://localhost:27017
MONGODB_LOGGING_COLLECTION_NAME=api_logs
```

**Файл `appsettings.json` (общие настройки):**

``` json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    }
  }
}
```

## 3. Реализация Extension-метода

Для чистоты кода в `Program.cs` вынеси логику настройки в отдельный метод расширения.

``` csharp
using Serilog;
using Serilog.Events;

public static class LoggingExtensions
{
    public static WebApplicationBuilder AddLoggingToMongoDb(this WebApplicationBuilder builder)
    {
        // Извлечение данных из конфигурации (подгруженных из .env)
        var databaseUrl = builder.Configuration["MONGODB_LOGGING_DATABASE_URL"] 
            ?? throw new InvalidOperationException("MongoDB URL is missing");
            
        var collectionName = builder.Configuration["MONGODB_LOGGING_COLLECTION_NAME"] 
            ?? "logs";

        builder.Host.UseSerilog((context, services, configuration) => configuration
            .ReadFrom.Configuration(context.Configuration)
            .WriteTo.MongoDBBson(
                databaseUrl: databaseUrl,
                collectionName: collectionName,
                restrictedToMinimumLevel: LogEventLevel.Information,
                // Ограничение размера коллекции (рекомендуется для логов)
                cappedMaxSizeMb: 1024, 
                cappedMaxDocuments: 50000
            )
            .Enrich.FromLogContext()
            .Enrich.WithMachineName()
            .Enrich.WithProperty("Application", "Wolfix.Server"));

        return builder;
    }
}
```

## 4. Инициализация в Program.cs

Настрой правильный цикл запуска приложения, чтобы поймать ошибки даже на этапе инициализации хоста.

``` csharp
using Serilog;

DotNetEnv.Env.Load();

// Начальный логгер для отслеживания старта
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateBootstrapLogger();

try
{
    Log.Information("Starting Web API...");

    var builder = WebApplication.CreateBuilder(args);
    
    // Подключаем наш MongoDB логгер
    builder.AddLoggingToMongoDb();

    var app = builder.Build();

    // Middleware для автоматического логирования HTTP-запросов
    app.UseSerilogRequestLogging(); 

    app.MapControllers();
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Host terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

## 5. Продвинутое использование: LogContext

Благодаря `.Enrich.FromLogContext()`, ты можешь добавлять специфические данные в логи внутри контроллеров.

``` csharp
[ApiController]
[Route("[controller]")]
public class OrdersController : ControllerBase
{
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(ILogger<OrdersController> logger) => _logger = logger;

    [HttpPost]
    public IActionResult Create(Order order)
    {
        // Использование LogContext добавит UserId во все логи внутри блока using
        using (Serilog.Context.LogContext.PushProperty("UserId", order.UserId))
        {
            _logger.LogInformation("Processing order creation for {@Order}", order);
            // ... логика
            return Ok();
        }
    }
}
```

_В MongoDB этот лог сохранится как документ, где `UserId` и объект `Order` будут отдельными полями, а не просто текстом._

## 6. Почему именно MongoDB?

1. **Structured Data:** Вы можете фильтровать логи по любому полю объекта (например, `Properties.UserId`).
    
2. **Capped Collections:** Автоматическое удаление старых записей без фрагментации диска.
    
3. **TTL Indexes:** Возможность настроить удаление логов через X дней штатными средствами БД.
    
4. **Scalability:** Если логов становится слишком много, MongoDB легче масштабировать, чем SQL-решения.
    

## 7. Чек-лист проверки в MongoDB Compass

1. Подключитесь к `mongodb://localhost:27017`.
    
2. Выберите базу данных (указанную в строке подключения).
    
3. Проверьте коллекцию `api_logs`.
    
4. Убедитесь, что поле `Timestamp` имеет тип `Date`, а `Properties` содержит ваши дополнительные данные.
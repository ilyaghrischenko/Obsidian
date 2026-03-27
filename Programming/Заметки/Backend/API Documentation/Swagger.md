tags: [#backend, #api-documentation, #swagger]

## Что такое Swagger?

**Swagger** в ASP.NET Core Web API — это набор инструментов для генерации интерактивной документации по HTTP API на основе OpenAPI-описания. На практике он чаще всего используется вместе с `Swashbuckle.AspNetCore`, чтобы автоматически показать все маршруты, параметры, модели ответа и возможность вызывать методы прямо из браузера.

Swagger особенно полезен в проектах на [[ASP.NET Core MVC]] и [[Minimal API]], где нужно быстро документировать эндпоинты и держать контракт API в актуальном состоянии.

## Зачем использовать?

- Показывает все доступные эндпоинты в одном месте.
- Позволяет тестировать API без Postman или другого клиента.
- Автоматически документирует параметры, коды ответов и схемы DTO.
- Упрощает поддержку авторизации через `Bearer`-токен и связку с [[JWT]].
- Хорошо сочетается с [[Versioning]], если API развивается по версиям.
- Убирает лишний шум из `Program.cs`, если вынести настройку в расширения наподобие [[WebApplicationBuilderExtensions]].

## Как это работает?

Swagger подключается в двух местах:

- через DI-регистрацию сервисов в `builder.Services`;
- через HTTP-конвейер, где подключаются [[Middleware]] `UseSwagger()` и `UseSwaggerUI()`.

### ✅ Базовая настройка в `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### ⚙️ Что делает каждая часть

- `AddEndpointsApiExplorer()` подготавливает метаданные эндпоинтов для Swagger.
- `AddSwaggerGen()` генерирует OpenAPI-документ.
- `UseSwagger()` отдает JSON-описание API.
- `UseSwaggerUI()` поднимает браузерный интерфейс, обычно по `/swagger`.

### ✅ Настройка через extension methods

Если проект разрастается, регистрацию лучше вынести в отдельный класс. Это хорошо сочетается с подходом из [[WebApplicationBuilderExtensions]] и сохраняет чистый `Program.cs`.

```csharp
public static class SwaggerExtensions
{
    public static IServiceCollection AddSwaggerDocumentation(this IServiceCollection services)
    {
        services.AddEndpointsApiExplorer();

        services.AddSwaggerGen(options =>
        {
            options.SwaggerDoc("v1", new OpenApiInfo
            {
                Title = "Shop API",
                Version = "v1",
                Description = "Документация для Web API"
            });
        });

        return services;
    }

    public static IApplicationBuilder UseSwaggerDocumentation(this IApplicationBuilder app)
    {
        app.UseSwagger();
        app.UseSwaggerUI(options =>
        {
            options.SwaggerEndpoint("/swagger/v1/swagger.json", "Shop API v1");
            options.RoutePrefix = "docs";
        });

        return app;
    }
}
```

Подключение:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddSwaggerDocumentation();

var app = builder.Build();

app.UseSwaggerDocumentation();
app.MapControllers();

app.Run();
```

### 🔐 Пример настройки авторизации Bearer

Если API защищен через [[JWT]], в Swagger UI удобно добавить поле для токена:

```csharp
services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Вставь JWT токен без префикса Bearer"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});
```

После этого в Swagger UI появится кнопка `Authorize`, через которую можно передавать токен для защищенных методов.

## Заключение

Swagger в ASP.NET Core Web API нужен для быстрой и наглядной документации API, ручного тестирования и поддержки актуального контракта. Для маленьких проектов достаточно базовой конфигурации в `Program.cs`, а для более крупных лучше выносить настройку в отдельные extension methods и комбинировать с [[DI injection types]], [[Middleware]] и [[Versioning]].

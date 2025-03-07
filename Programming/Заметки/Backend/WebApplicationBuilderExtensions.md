## Введение

`WebApplicationBuilderExtensions` — это статический класс, предназначенный для расширения возможностей `WebApplicationBuilder` ([[Builder]]) в ASP.NET Core. Он помогает вынести регистрацию сервисов, [[Middleware]] и конфигураций в отдельные методы, упрощая `Program.cs`.

## Использование

Добавьте пространство имен и вызовите необходимые методы расширения внутри `Program.cs`.

```csharp
using StoronnimV.Api.Extensions;

var builder = WebApplication.CreateBuilder(args);

builder.AddApplicationServices()
       .AddIntegrationServices()
       .AddRepositories()
       .AddOptions()
       .AddPooledDbContextFactory()
       .AddFluentValidation()
       .AddSerilogLogger()
       .AddAutoMapper()
       .AddCors()
       .AddHangfire()
       .AddJwtBearer()
       .AddResponseCompression()
       .AddRateLimiter();

var app = builder.Build();
```

## Основные методы

### 1. `AddApplicationServices`

Регистрирует основные сервисы приложения ([[DI injection types]]):

```csharp
builder.Services.AddScoped<INewsService, NewsService>();
builder.Services.AddScoped<IScheduleService, ScheduleService>();
```

### 2. `AddIntegrationServices`

Добавляет сервисы, взаимодействующие с внешними системами ([[DI injection types]]):

```csharp
builder.Services.AddScoped<ScheduleStatusUpdaterService>();
```

### 3. `AddRepositories`

Регистрирует репозитории для работы с базой данных ([[DI injection types]]):

```csharp
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```

### 4. `AddOptions`

Настраивает конфигурационные параметры ([[IOptions]]):

```csharp
builder.Services.Configure<JwtOptions>(builder.Configuration.GetSection("JwtOptions"));
```

### 5. `AddPooledDbContextFactory`

Добавляет поддержку `DbContextFactory` с PostgreSQL ([[Entity Framework Core]]):

```csharp
builder.Services.AddPooledDbContextFactory<StoronnimVContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("CloudConnection")));
```

### 6. `AddFluentValidation`

Регистрирует валидаторы ([[Fluent Validation]]):

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<LogInRequestValidator>();
```

### 7. `AddSerilogLogger`

Настраивает логирование с помощью [[Serilog]]:

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();
```

### 8. `AddAutoMapper`

Настраивает [[AutoMapper]]:

```csharp
builder.Services.AddAutoMapper(typeof(NewsMappingProfile).Assembly);
```

### 9. `AddCors`

Разрешает [[CORS]] для [[React]]-приложения:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReactApp", policy =>
        policy.WithOrigins("http://localhost:5173")
              .AllowAnyHeader()
              .AllowAnyMethod());
});
```

### 10. `AddHangfire`

Добавляет поддержку [[Hangfire]] для фоновых задач:

```csharp
builder.Services.AddHangfire(config => config.UsePostgreSqlStorage(options =>
    options.UseNpgsqlConnection(builder.Configuration.GetConnectionString("CloudConnection"))));
```

### 11. `AddJwtBearer`

Настраивает [[JWT]]-аутентификацию:

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = jwtOptions.GetKey()
        };
    });
```

### 12. `AddResponseCompression`

Добавляет сжатие Brotli и Gzip ([[Response Compression]]):

```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});
```

### 13. `AddRateLimiter`

Настраивает ограничение запросов ([[Rate Limiter]]):

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("DefaultPolicy", httpContext =>
        RateLimitPartition.GetFixedWindowLimiter("DefaultPolicy", _ =>
            new FixedWindowRateLimiterOptions { PermitLimit = 100, Window = TimeSpan.FromMinutes(1) }));
});
```

## Заключение

`WebApplicationBuilderExtensions` помогает сделать код чище, уменьшая размер `Program.cs` и упрощая конфигурацию сервиса через цепочку методов расширения.
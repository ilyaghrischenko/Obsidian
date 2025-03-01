## Что такое RateLimiter и зачем он нужен?
`RateLimiter` — это механизм ограничения частоты запросов к серверу. Он помогает предотвратить перегрузку системы, злоупотребление API и атаки типа DoS (Denial of Service).

Основные преимущества:
- Защита от перегрузки сервера.
- Контроль за использованием ресурсов.
- Улучшение производительности приложения.
- Предотвращение злоупотребления API.

## Подключение RateLimiter в ASP.NET Core
### Добавление RateLimiter через расширение
```csharp
public static WebApplicationBuilder AddRateLimiter(this WebApplicationBuilder builder)
{
    var rateLimiterOptions = builder.Configuration.GetSection("RateLimiterOptions")
        .Get<Options.RateLimiter.RateLimiterOptions>();

    if (rateLimiterOptions == null)
    {
        throw new KeyNotFoundException("RateLimiterOptions are not configured correctly.");
    }
    
    builder.Services.AddRateLimiter(options =>
    {
        rateLimiterOptions.Policies.ForEach(policy =>
        {
            AddLimiterPolicy(options, policy.PolicyName, policy.Limit, policy.Expiration);
        });
        
        options.RejectionStatusCode = rateLimiterOptions.StatusCode;
    });

    return builder;
}

private static void AddLimiterPolicy(RateLimiterOptions options, string policyName, int limit, TimeSpan expiration)
{
    options.AddPolicy(policyName, httpContext =>
        RateLimitPartition.GetFixedWindowLimiter(policyName, _ => new FixedWindowRateLimiterOptions
        {
            PermitLimit = limit,
            Window = expiration
        })
    );
}
```

## Конфигурация RateLimiter в `appsettings.json` [[IOptions]]
```json
{
  "RateLimiterOptions": {
    "StatusCode": 429,
    "Policies": [
      {
        "PolicyName": "DefaultPolicy",
        "Limit": 10,
        "Expiration": "00:01:00"
      }
    ]
  }
}
```

## Подключение в Program.cs

``` csharp
app.UseResponseCompression();
```

Здесь происходит вставка [[Middleware]] в наш [[ASP.NET Core Pipeline]]

## Итог

RateLimiter — это эффективный инструмент для контроля нагрузки на сервер. Использование политики ограничения запросов помогает защитить приложение от чрезмерной нагрузки и злоумышленников, улучшая стабильность и производительность API.
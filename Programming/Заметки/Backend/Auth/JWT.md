## 1. Что это и зачем?

**JWT (JSON Web Token)** — это стандарт открытого обмена данными, который используется для безопасной передачи информации между клиентом и сервером в виде токенов. Токены JWT часто используются в системах аутентификации и авторизации, так как они позволяют удостовериться в подлинности пользователя и передавать информацию о пользователе или его правах.

**Зачем использовать JWT?**
- **Безопасность**: JWT позволяет надежно передавать данные между сервером и клиентом с использованием криптографических механизмов (подпись, шифрование).
- **Отсутствие состояния**: Сервер не хранит состояние сессии. Все необходимые данные (например, информация о пользователе) содержатся непосредственно в токене.
- **Масштабируемость**: JWT удобен для микросервисных архитектур, так как токены могут быть переданы между различными сервисами.

JWT состоит из трех частей:
1. **Header** — заголовок, содержащий информацию о типе токена и алгоритме подписи.
2. **Payload** — полезная нагрузка, в которой находятся данные (например, информация о пользователе).
3. **Signature** — подпись, которая подтверждает целостность токена и проверяет его подлинность.

## 2. Как подключать JWT в ASP.NET Core

Для настройки JWT аутентификации и авторизации в ASP.NET Core необходимо выполнить следующие шаги.

### Пример подключения:

### **Чтение конфигурации и настройка `JwtOptions`**:

Тут мы пользуемся паттерном [[IOptions]]

```csharp
JwtOptions? jwtOptions = builder.Configuration.GetSection("JwtOptions").Get<JwtOptions>();

if (jwtOptions == null)
{
    throw new KeyNotFoundException("JwtOptions are not configured correctly.");
}

builder.Services.Configure<JwtOptions>(builder.Configuration.GetSection("JwtOptions"));
```

### **Настройка аутентификации с использованием JWT**:

через option.Events мы можем с каждым запросом на метод контроллера, который требует Авторизации вытягивать [[HttpOnly Cookie]]

``` csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.RequireHttpsMetadata = false;
        options.TokenValidationParameters = new()
        {
            ValidateIssuer = true,
            ValidIssuer = jwtOptions.ISSUER,

            ValidateAudience = true,
            ValidAudience = jwtOptions.AUDIENCE,

            ValidateLifetime = true,

            ValidateIssuerSigningKey = true,
            IssuerSigningKey = jwtOptions.GetKey()
        };

        options.Events = new JwtBearerEvents
        {
            OnMessageReceived = context =>
            {
                string? token = context.Request.Cookies["Token"];
                if (!string.IsNullOrEmpty(token))
                {
                    context.Token = token;
                }

                return Task.CompletedTask;
            }
        };
    });
```

### **Настройка авторизации с использованием ролей**:

Таким образом мы можем создать [[JWT Policy]]

``` csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("SuperAdminOnly", policy =>
        policy.RequireRole("SuperAdmin"));
```

### **Настройка Swagger для работы с JWT**:

``` csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme()
    {
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.Http,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        Description =
            "Input your JWT token in the 'Authorization' header like this: \"Authorization: Bearer {yourJWT}\""
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

В данном примере:

- Мы читаем конфигурацию из файла и настраиваем параметры для JWT.
- Добавляем аутентификацию с использованием JWT с указанием параметров валидации токена (например, проверка подписей и аудитории).
- Настроили Swagger для работы с JWT, чтобы можно было легко тестировать защищенные API эндпоинты с использованием токенов.
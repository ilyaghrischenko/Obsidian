## **🔍 Что такое FastEndpoints**

  

[FastEndpoints](https://fast-endpoints.com/) — это библиотека для ASP.NET Core, которая упрощает создание веб-API и делает их быстрее, чище и более структурированными по сравнению с обычными контроллерами или минимальными API.

  

Она предлагает мощную и лаконичную альтернативу привычным Controller-базированным архитектурам, предоставляя:

- Автоматическую маршрутизацию
    
- Валидацию через FluentValidation
    
- Встроенные фильтры
    
- Асинхронные обработчики запросов
    
- Замену MediatR через встроенные интерфейсы ICommand / ICommandHandler
    

---

## **🚀 Почему FastEndpoints**

- ✅ Меньше кода, больше производительности
    
- ✅ Без атрибутов и магии — все строго типизировано
    
- ✅ Встроенная валидация, авторизация, фильтрация
    
- ✅ Отличная интеграция с Swagger
    
- ✅ Поддержка CQRS из коробки
    

---

## **🧠 Архитектура запроса (Endpoint)**

``` csharp
public class CreateUserEndpoint : Endpoint<CreateUserRequest, CreateUserResponse>
{
    public override async Task HandleAsync(CreateUserRequest req, CancellationToken ct)
    {
        // обработка запроса
        var user = new User { Name = req.Name };
        await SaveToDb(user);

        await SendAsync(new CreateUserResponse { Id = user.Id });
    }
}
```

FastEndpoints автоматически связывает маршруты по именам классов (или по вашим настройкам).

---

## **🧾 Пример запроса и ответа**

``` csharp
public class CreateUserRequest
{
    public string Name { get; set; }
}

public class CreateUserResponse
{
    public Guid Id { get; set; }
}
```

## **🧩 Валидация (FluentValidation)**

``` csharp
public class CreateUserValidator : Validator<CreateUserRequest>
{
    public CreateUserValidator()
    {
        RuleFor(x => x.Name).NotEmpty().WithMessage("Имя обязательно");
    }
}
```

Validator будет применён автоматически, если его тип совпадает с типом запроса.

---

## **🔄 Замена [[CQRS + Mediator]] через** 

## **ICommand**

##  **/** 

## **ICommandHandler**

  

FastEndpoints поддерживает **встроенные интерфейсы для CQRS**, которые могут полностью **заменить MediatR**:

``` csharp
public class CreateUserCommand : ICommand<CreateUserResponse>
{
    public string Name { get; set; }
}

public class CreateUserHandler : ICommandHandler<CreateUserCommand, CreateUserResponse>
{
    public async Task<CreateUserResponse> ExecuteAsync(CreateUserCommand cmd, CancellationToken ct)
    {
        // логика создания пользователя
        var id = Guid.NewGuid();
        return new CreateUserResponse { Id = id };
    }
}
```

Регистрация в DI не требуется — FastEndpoints всё поднимает автоматически. В Endpoint можно вызывать:

``` csharp
await SendAsync(cmd); // или
var response = await cmd.ExecuteAsync();
```

## **🔐 Авторизация**

``` csharp
public class SecureEndpoint : EndpointWithoutRequest
{
    public override void Configure()
    {
        Verbs(Http.GET);
        Routes("/secure");
        Roles("Admin"); // авторизация по ролям
    }

    public override async Task HandleAsync(CancellationToken ct)
    {
        await SendAsync("Secure content");
    }
}
```

## **📦 Установка**

``` csharp
dotnet add package FastEndpoints
dotnet add package FastEndpoints.Swagger
dotnet add package FluentValidation
```

В Program.cs:

``` csharp
builder.Services.AddFastEndpoints();
builder.Services.AddSwaggerDoc();

var app = builder.Build();

app.UseFastEndpoints();
app.UseSwaggerGen();
```

## **📌 Особенности**
|**Возможность**|**Поддерживается FastEndpoints**|
|---|---|
|Swagger|✅ Да|
|Валидация|✅ Через FluentValidation|
|CQRS|✅ Своими интерфейсами|
|Фильтры|✅ До и после запроса|
|Авторизация|✅ По ролям и политикам|
|Асинхронность|✅ Полная поддержка|
|Поддержка DI|✅ Встроенная|

---

## **🧪 Заключение**

  

FastEndpoints — это современный, производительный и удобный способ построения API в .NET. Он идеально подходит, если ты хочешь:

- избавиться от громоздких контроллеров
    
- использовать CQRS без [[Mediator]]
    
- писать лаконичный и быстрый код
    

  

📘 **Официальный сайт:** [fast-endpoints.com](https://fast-endpoints.com/)

📘 **Документация:** [docs.fast-endpoints.com](https://docs.fast-endpoints.com)
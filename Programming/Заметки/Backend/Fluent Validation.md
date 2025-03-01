## 1. Что это такое?

FluentValidation — это библиотека для ASP.NET Core, которая предоставляет удобный и мощный способ для реализации валидации данных в приложении. Она позволяет описывать правила валидации с использованием fluent API (потокового API), делая код легко читаемым и поддерживаемым.

С помощью FluentValidation можно:
- Определять правила валидации для моделей или DTO.
- Легко расширять и добавлять новые правила валидации.
- Интегрировать с ASP.NET Core для автоматической валидации запросов.

## 2. Какой NuGet пакет использовать?

Для того чтобы использовать FluentValidation в проекте, нужно установить NuGet пакет:

```bash
dotnet add package FluentValidation
```

Это основной пакет, который предоставляет функциональность для валидации.

## 3. Подключение в ASP.NET Core

После установки пакета, необходимо настроить и подключить FluentValidation в проект.

Добавьте следующие строки в метод `ConfigureServices` в файле `Program.cs` или `Startup.cs`:

``` csharp
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<LogInRequestValidator>();
builder.Services.AddValidatorsFromAssemblyContaining<EditBasicAdminLoginRequestValidator>();
builder.Services.AddValidatorsFromAssemblyContaining<EditBasicAdminPasswordRequestValidator>();
```

Эти строки подключают автоматическую валидацию и добавляют валидаторы из указанных сборок. Вы можете добавлять валидаторы из других сборок аналогичным образом.

## 4. Примеры валидаторов

### Пример 1: Валидация для логина

Предположим, у вас есть класс `LogInRequest`, и вы хотите проверить, чтобы его поля были корректными.

``` csharp
public class LogInRequest
{
    public string Username { get; set; }
    public string Password { get; set; }
}

public class LogInRequestValidator : AbstractValidator<LogInRequest>
{
    public LogInRequestValidator()
    {
        RuleFor(x => x.Username)
            .NotEmpty().WithMessage("Username is required")
            .Length(5, 20).WithMessage("Username must be between 5 and 20 characters");

        RuleFor(x => x.Password)
            .NotEmpty().WithMessage("Password is required")
            .Length(8, 50).WithMessage("Password must be between 8 and 50 characters");
    }
}
```

В этом примере:

- Мы проверяем, что `Username` не пустой и его длина находится в пределах от 5 до 20 символов.
- Мы также проверяем, что `Password` не пустой и его длина от 8 до 50 символов.

### Пример 2: Валидация для редактирования логина администратора

``` csharp
public class EditBasicAdminLoginRequest
{
    public string NewLogin { get; set; }
}

public class EditBasicAdminLoginRequestValidator : AbstractValidator<EditBasicAdminLoginRequest>
{
    public EditBasicAdminLoginRequestValidator()
    {
        RuleFor(x => x.NewLogin)
            .NotEmpty().WithMessage("New login is required")
            .Matches("^[a-zA-Z0-9]*$").WithMessage("Login must contain only letters and numbers");
    }
}
```

В этом примере:

- Мы проверяем, что новый логин не пустой и соответствует регулярному выражению (содержит только буквы и цифры).

## Заключение

FluentValidation предоставляет мощный способ для выполнения валидации в ASP.NET Core. Использование этой библиотеки позволяет вам писать понятный, чистый и легко поддерживаемый код для проверки данных.

4o mini
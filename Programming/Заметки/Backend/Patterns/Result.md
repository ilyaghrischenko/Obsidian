## Что такое Result Pattern?

**Result Pattern** — это шаблон проектирования, используемый для обработки результатов операций, которые могут завершиться как успешно, так и с ошибкой. Он помогает избежать использования исключений для управления потоком выполнения и делает код более предсказуемым и безопасным.

Вместо выбрасывания исключений при ошибке, операция возвращает объект типа `Result<T>`, содержащий либо результат, либо сообщение об ошибке и статус.

---

## Пример реализации

```csharp
public sealed class Result<TValue>
{
    public TValue? Value { get; }
    public string? ErrorMessage { get; }
    public bool IsSuccess => ErrorMessage == null;
    public HttpStatusCode StatusCode { get; }

    private Result(TValue value, HttpStatusCode statusCode)
    {
        Value = value;
        ErrorMessage = null;
        StatusCode = statusCode;
    }

    private Result(string errorMessage, HttpStatusCode statusCode)
    {
        Value = default;
        ErrorMessage = errorMessage;
        StatusCode = statusCode;
    }

    public static Result<TValue> Success(TValue value, HttpStatusCode statusCode = HttpStatusCode.OK)
        => new(value, statusCode);

    public static Result<TValue> Failure(string errorMessage, HttpStatusCode statusCode = HttpStatusCode.BadRequest)
        => new(errorMessage, statusCode);

    public TResult Map<TResult>(Func<TValue, TResult> onSuccess, Func<string, TResult> onFailure)
    {
        return IsSuccess ? onSuccess(Value!) : onFailure(ErrorMessage!);
    }
}
```

---

## Зачем нужен Result Pattern

### ✅ Преимущества:

- **Предсказуемость**: все возможные исходы явно описаны.
    
- **Безопасность**: меньше шансов забыть обработать ошибку.
    
- **Простота тестирования**: можно легко проверить оба исхода.
    
- **Улучшение читаемости**: вызов `.Map()` позволяет декларативно описывать поведение.
    

### ❌ Недостатки:

- Дополнительный код по сравнению с обычными исключениями.
    
- Может усложнить цепочки вызовов, если не использовать методы-обертки (например, `.Bind()`, `.Map()`, `.Match()` и т.д.)
    

---

## Примеры использования

### Успешный результат

```csharp
Result<string> result = Result<string>.Success("OK");
if (result.IsSuccess)
{
    Console.WriteLine(result.Value);
}
```

### Ошибочный результат

```csharp
Result<string> result = Result<string>.Failure("Something went wrong", HttpStatusCode.InternalServerError);
if (!result.IsSuccess)
{
    Console.WriteLine(result.ErrorMessage);
}
```

### Использование Map

```csharp
var response = GetUser(id)
    .Map(
        user => $"Пользователь: {user.Name}",
        error => $"Ошибка: {error}"
    );

Console.WriteLine(response);
```

---

## Почему это лучше, чем исключения?

Исключения — это механизм для **непредвиденных ошибок**. Если ошибка предсказуема и часть бизнес-логики (например, "пользователь не найден"), лучше использовать `Result<T>`, а не `throw`.

---

## Заключение

**Result Pattern** делает код более читаемым, предсказуемым и безопасным. Он особенно полезен в многоуровневой бизнес-логике, API-ответах и функциональном программировании. Если операция может завершиться неудачно — лучше использовать `Result<T>`, чем полагаться на исключения.
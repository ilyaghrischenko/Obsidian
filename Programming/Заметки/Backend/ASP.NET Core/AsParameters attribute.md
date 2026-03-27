tags: [#backend, #asp-net-core, #asparameters]

## Что такое `[AsParameters]`
`[AsParameters]` — это атрибут для Minimal APIs, который указывает фреймворку собрать параметры HTTP-запроса (маршрут, строку запроса, заголовки) и упаковать их в один типизированный объект. 

По умолчанию Minimal API пытается привязать сложные объекты из тела запроса (`[FromBody]`). `[AsParameters]` меняет это поведение, заставляя биндер сканировать метаданные свойств самого объекта.

## Для чего использовать
1. **Борьба с раздутыми сигнатурами:** Сокращение количества аргументов в обработчике эндпоинта.
2. **Смешанный биндинг:** Инкапсуляция параметров из разных источников (`Route`, `Query`, `Header`) в единую логическую структуру.
3. **Переиспользование:** Возможность использовать одни и те же наборы параметров (например, для пагинации) в разных эндпоинтах.

### Пример: Смешанный биндинг

```csharp
using Microsoft.AspNetCore.Mvc;

namespace App.Features.Documents;

// 1. Определение параметров с указанием их источников
public readonly record struct GetDocumentParams(
    [FromRoute] Guid DocumentId,
    [FromHeader(Name = "X-Tenant-Id")] string TenantId,
    [FromQuery] bool IncludeDetails = false
);

// 2. Использование в эндпоинте
public static class GetDocumentEndpoint
{
    public static void Map(IEndpointRouteBuilder app)
    {
        app.MapGet("/api/docs/{DocumentId}", Handle);
    }

    private static IResult Handle([AsParameters] GetDocumentParams request)
    {
        // Доступ к данным из разных частей HTTP-запроса через один объект
        var id = request.DocumentId;
        var tenant = request.TenantId;
        
        return Results.Ok();
    }
}
```

## Правила композиции: `[AsParameters]` и `[FromBody]`

Технически фреймворк позволяет поместить свойство с атрибутом `[FromBody]` внутрь структуры `[AsParameters]`, но **делать так не следует**. Это нарушает семантику HTTP-запроса и усложняет чтение кода.

* **`[AsParameters]`** предназначен строго для метаданных и параметров окружения: `[FromRoute]`, `[FromQuery]`, `[FromHeader]`.
* **`[FromBody]`** предназначен для полезной нагрузки (Payload), которая десериализуется из JSON в тяжелый DTO (ссылочный тип `record` или `class`).

Они должны жить в сигнатуре метода раздельно:

```csharp
// Правильное разделение: параметры маршрута/заголовков отдельно от тела (JSON)
private static async Task<Ok> Handle(
    [AsParameters] UpdateUserRouteParams routeParams, 
    [FromBody] UpdateUserDto payload,                 
    [FromServices] DbContext db)
{ ... }
```

## Стратегия применения в Vertical Slice Architecture (VSA)

В [[Vertical Sliced Architecture (VSA)]] изоляция фичей важнее переиспользования кода. Использование `[AsParameters]` зависит от назначения параметров и их количества. Придерживайся трех правил:

### 1. Вынесение в Common (Общие механизмы)

Параметры, не связанные с конкретной бизнес-логикой (пагинация, фильтрация по датам), **нужно** выносить в общие папки (например, `Features/Common/Parameters`) для переиспользования во всех слайсах.

``` csharp
// Features/Common/Parameters/PaginationParams.cs
public readonly record struct PaginationParams(
    [FromQuery] int Page = 1,
    [FromQuery] int PageSize = 30
);
```

### 2. Инкапсуляция внутри слайса (Специфичные запросы)

Если эндпоинт принимает комбинацию параметров маршрута и специфичных фильтров строки запроса (3 и более параметров в сумме), создавай структуру `[AsParameters]` **прямо внутри статического класса фичи**. Не пытайся переиспользовать ее в других местах.

``` csharp
// Features/Users/GetUserProfile.cs
public static class GetUserProfile
{
    // Структура живет внутри фичи и обслуживает только этот эндпоинт
    public readonly record struct RequestParams(
        [FromRoute] Guid UserId,
        [FromHeader(Name = "X-Tenant-Id")] string TenantId,
        [FromQuery] bool IncludeActivityHistory = false
    );

    // Minimal APIs отлично умеет биндить несколько [AsParameters] одновременно
    public static async Task<Ok<Response>> Handle(
        [AsParameters] RequestParams request,
        [AsParameters] PaginationParams pagination,
        [FromServices] DbContext db) 
    { ... }
}
```

### 3. Отказ от `[AsParameters]` (Простые запросы)

Не используй структуры для 1-2 параметров. Если эндпоинт принимает только один ID из маршрута или один поисковый запрос, передавай их напрямую в метод. Создание структуры ради одного свойства — это оверинжиниринг.

``` csharp
// Правильно: без лишних оберток
private static async Task<Ok<Response>> Handle(
    [FromRoute] Guid userId, 
    [FromServices] DbContext db)
{ ... }
```

## `readonly record struct` vs `record` (class)

Выбор между структурой (Value Type) и классом (Reference Type) зависит от размера данных и их содержимого. Главная цель структур в Web API — снижение нагрузки на сборщик мусора (GC) за счет аллокации на стеке.

### Когда использовать `readonly record struct`

Используй по умолчанию для `[AsParameters]`, если выполняются следующие условия:

- **Малый размер:** Структура содержит мало полей (условно до 4-6) и весит до 24-32 байт.
    
- **Примитивные типы:** Внутри только value types (`int`, `Guid`, `bool`, `DateTime`, `enum`).
    

**Почему:** Структура живет на стеке и уничтожается сразу после выхода из метода. Нулевая нагрузка на GC. Передача по значению (копирование) нескольких байт для процессора происходит мгновенно.

``` csharp
// Идеальный кандидат для struct
public readonly record struct PaginationParams(  
    [FromQuery, Range(1, int.MaxValue)] int Page = 1,  
    [FromQuery, Range(1, 100)] int PageSize = 30  
);
```

### Когда использовать `record` (ссылочный тип)

Используй обычный `record`, если параметры не подходят под критерии структуры:

- **Наличие ссылочных типов:** Если параметры содержат `string`, массивы или коллекции (`string[]`, `List<T>`). Аллокация этих полей всё равно произойдет в куче, поэтому оборачивать их в `struct` бессмысленно — GC всё равно придется работать, а процессор будет тратить такты на копирование структуры при передаче.
    
- **Большой размер:** Много параметров (10+). Копирование тяжелой структуры по значению при передаче между слоями начнет бить по CPU.
    
- **DTO из тела запроса:** Для `[FromBody]` всегда используй классы/рекорды, так как JSON-десериализатор оптимизирован под ссылочные типы.

``` csharp
// Правильный кандидат для record (class)
public sealed record ComplexSearchParams(
    [FromQuery] string SearchTerm,          // string - ссылочный тип
    [FromQuery] string[] Tags,              // массив - ссылочный тип
    [FromQuery] Guid? CategoryId,
    [FromQuery] int Page = 1
);
```

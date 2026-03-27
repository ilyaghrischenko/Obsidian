tags: [#backend, #pagination, #cursor-pagination]

## Что такое Cursor-based Pagination?

**Cursor-based pagination** - это способ пагинации, при котором следующая страница строится не через номер страницы и `OFFSET`, а через специальный курсор. Обычно курсором выступает уникальное или монотонно возрастающее поле, например `Id` или `CreatedAt`.

Этот подход особенно полезен для API с большими объемами данных, бесконечной прокруткой и лентами событий. Часто используется в API на [[ASP.NET Core MVC]], [[Minimal API]] и в системах, где важна стабильная выборка.

## Зачем использовать?

- Лучше масштабируется на больших таблицах.
- Не заставляет БД пропускать тысячи строк через `OFFSET`.
- Дает более стабильный результат при частых вставках и удалениях.
- Удобен для infinite scroll и мобильных клиентов.
- Хорошо подходит для коллекций, отсортированных по `Id`, `CreatedAt` или другому индексу.

## Как это работает?

Клиент передает курсор последнего элемента с предыдущей страницы. Сервер использует его как точку продолжения выборки.

### ✅ Пример с `Id`

```csharp
var cursor = 120;
var pageSize = 10;

var users = await dbContext.Users
    .Where(x => x.Id > cursor)
    .OrderBy(x => x.Id)
    .Take(pageSize)
    .ToListAsync();
```

В ответе сервер обычно возвращает:

- список элементов;
- `nextCursor` для следующего запроса.

### ✅ Пример ответа API

```csharp
return Results.Ok(new
{
    items = users,
    nextCursor = users.LastOrDefault()?.Id
});
```

### ⚙️ Пример endpoint

```csharp
app.MapGet("/users", async (long? cursor, int pageSize, AppDbContext dbContext) =>
{
    var query = dbContext.Users
        .OrderBy(x => x.Id)
        .AsQueryable();

    if (cursor is not null)
    {
        query = query.Where(x => x.Id > cursor.Value);
    }

    var items = await query
        .Take(pageSize)
        .Select(x => new UserDto(x.Id, x.Name))
        .ToListAsync();

    var nextCursor = items.LastOrDefault()?.Id;

    return Results.Ok(new
    {
        items,
        nextCursor
    });
});
```

### ⚙️ Практические замечания

- Поле курсора должно быть уникальным или почти уникальным.
- Сортировка должна быть стабильной и одинаковой между запросами.
- Для `CreatedAt` часто добавляют второй ключ, например `Id`, чтобы избежать неоднозначности.
- В NoSQL-хранилищах и при работе с [[MongoDB.Driver в ASP.NET Core Web API]] такой подход тоже встречается очень часто.

### Основные минусы

- Реализация сложнее, чем у [[Offset]].
- Нельзя удобно прыгать сразу на страницу `25`.
- Нужно аккуратно проектировать контракт ответа и формат курсора.

## Заключение

**Cursor-based pagination** лучше подходит для нагруженных API и больших наборов данных. Она дает более предсказуемую производительность и стабильную выдачу, особенно если обычный `OFFSET` в [[Offset]] уже начинает тормозить.

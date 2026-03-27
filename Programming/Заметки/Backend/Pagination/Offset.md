tags: [#backend, #pagination, #offset-pagination]

## Что такое Offset Pagination?

**Offset pagination** - это способ постраничного получения данных, при котором база данных пропускает определенное количество строк через `OFFSET` и затем возвращает следующую порцию через `LIMIT` или `Take`.

Такой подход часто встречается в API на [[ASP.NET Core MVC]], [[Minimal API]] и при работе с [[Dapper]] или LINQ-запросами через [[LINQ]].

## Зачем использовать?

- Простой и понятный механизм пагинации.
- Удобно реализуется через параметры `page` и `pageSize`.
- Подходит для таблиц, где нужен быстрый старт без сложной логики.
- Легко интегрируется в HTTP API и UI со страницами `1, 2, 3...`.
- Хорошо читается в коде и SQL.

## Как это работает?

Идея простая: вычисляется количество записей, которые нужно пропустить.

Формула:

```csharp
var offset = (page - 1) * pageSize;
```

### ✅ Пример с LINQ

```csharp
var page = 3;
var pageSize = 10;

var users = await dbContext.Users
    .OrderBy(x => x.Id)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

### ✅ Пример с SQL / Dapper

```csharp
var page = 3;
var pageSize = 10;
var offset = (page - 1) * pageSize;

const string sql = """
SELECT Id, Name, Email
FROM Users
ORDER BY Id
OFFSET @Offset ROWS FETCH NEXT @PageSize ROWS ONLY
""";

var users = await connection.QueryAsync<UserDto>(sql, new
{
    Offset = offset,
    PageSize = pageSize
});
```

### ⚙️ Пример в Web API

```csharp
app.MapGet("/users", async (int page, int pageSize, AppDbContext dbContext) =>
{
    var items = await dbContext.Users
        .OrderBy(x => x.Id)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .Select(x => new UserDto(x.Id, x.Name))
        .ToListAsync();

    return Results.Ok(items);
});
```

### Основные минусы

- На больших таблицах работает хуже, потому что БД все равно должна пропустить много строк.
- При изменении данных между запросами возможны дубли и пропуски записей.
- Не лучший выбор для бесконечной прокрутки и больших объемов данных.

Если нужна более стабильная выдача для больших коллекций, чаще используют [[Cursor-based]].

## Заключение

**Offset pagination** хорошо подходит для простой постраничной навигации и стандартных CRUD API. Это удобный стартовый вариант, но на больших объемах данных он проигрывает по производительности и стабильности подходу [[Cursor-based]].

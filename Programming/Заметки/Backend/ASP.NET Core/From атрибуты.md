В ASP.NET Core для обработки HTTP-запросов используется механизм привязки параметров к данным из различных источников. Атрибуты помогают указать, откуда именно нужно брать данные. Ниже приведены основные и часто используемые атрибуты.

---

## 1. `[FromRoute]`

**Назначение:** Извлекает данные из параметров маршрута (URL).

**Пример маршрута:** `/users/{id}`

**Пример использования:**

```csharp
[HttpGet("users/{id}")]
public IActionResult GetUser([FromRoute] int id)
{
    // Использует id из URL
    return Ok($"User ID: {id}");
}
```

---

## 2. `[FromQuery]`

**Назначение:** Извлекает данные из строки запроса (query string).

**Пример запроса:** `/users/search?name=John`

**Пример:**

```csharp
[HttpGet("users/search")]
public IActionResult SearchUser([FromQuery] string name)
{
    return Ok($"Search by name: {name}");
}
```

---

## 3. `[FromBody]`

**Назначение:** Извлекает данные из тела запроса (обычно JSON).

**Обычно используется с POST/PUT.**

**Пример:**

```csharp
public class UserDto
{
    public string Name { get; set; }
    public int Age { get; set; }
}

[HttpPost("users")]
public IActionResult CreateUser([FromBody] UserDto user)
{
    return Ok($"Created user: {user.Name}");
}
```

---

## 4. `[FromForm]`

**Назначение:** Извлекает данные из отправленной формы (`application/x-www-form-urlencoded` или `multipart/form-data`).

Часто используются при получении модели, которая содержит IFormFile(фото/видео)

**Пример:**

```csharp
[HttpPost("upload")]
public IActionResult Upload([FromForm] string description, [FromForm] IFormFile file)
{
    // Обрабатывает файл и описание из формы
    return Ok();
}
```

---

## 5. `[FromHeader]`

**Назначение:** Извлекает данные из HTTP-заголовков.

**Пример:**

```csharp
[HttpGet("secure")]
public IActionResult GetSecureData([FromHeader(Name = "X-Api-Key")] string apiKey)
{
    return Ok($"Header value: {apiKey}");
}
```

---

## 6. `[FromServices]`

**Назначение:** Извлекает зависимость из контейнера внедрения зависимостей (DI).

**Пример:**

```csharp
[HttpGet("check")]
public IActionResult CheckService([FromServices] IMyService service)
{
    return Ok(service.GetStatus());
}
```

---

## 7. Без атрибута (по умолчанию)

Если атрибут не указан:

- Примитивы по умолчанию берутся из **маршрута или query string**.
    
- Complex types (объекты) — из **тела запроса (body)**.
    

**Пример:**

```csharp
[HttpPost("login")]
public IActionResult Login(LoginDto dto) // будет [FromBody] по умолчанию
{
    return Ok();
}
```

---

## Практические советы:

- **[FromBody]** можно использовать только один раз на метод.
    
- Комбинируйте атрибуты: `[FromRoute]`, `[FromQuery]`, `[FromBody]`, `[FromHeader]` и др.
    
- Указывайте атрибут явно, чтобы не полагаться на поведение по умолчанию.
    

---

## Другие полезные атрибуты:

- `[BindRequired]` — генерирует ошибку, если привязка не удалась.
    
- `[BindNever]` — исключает свойство из привязки.
    
- `[ModelBinder]` — кастомная привязка с использованием собственного биндера.
    
- `[ValidateNever]` — исключает свойство из валидации.
    

---

Этот набор атрибутов охватывает 95% всех типичных случаев в ASP.NET Core приложениях. Используйте их осознанно для ясного и предсказуемого поведения API.
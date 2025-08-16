## **Введение**


Minimal API — это упрощённый способ создания HTTP API в ASP.NET Core без необходимости использования контроллеров и атрибутов. Такой подход особенно удобен для небольших сервисов, микросервисов или быстрых прототипов.
  

**Ключевые преимущества:**

- Меньше шаблонного кода
    
- Простая регистрация маршрутов
    
- Высокая производительность
    
- Легко начать и быстро развивать
    

---

## **Пример минимального API**

``` csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello, World!");

app.Run();
```

---

## **Поддержка DI (Dependency Injection)**

  

Minimal API полностью поддерживает DI. Параметры в делегате маршрута можно получать напрямую из контейнера зависимостей:

``` csharp
app.MapGet("/users", (IUserService userService) =>
{
    return userService.GetAllUsers();
});
```

---

## **Возврат результатов**

  

Minimal API позволяет возвращать:

- **Простые типы** (string, int, bool)
    
- **DTO/объекты** (автоматическая сериализация в JSON)
    
- **IResult** (для полного контроля над ответом)
    

``` csharp
app.MapGet("/status", () => Results.Ok(new { status = "ok" }));
```

---

## **Правильная структура проекта с Minimal API**

  

Чтобы проект был масштабируемым и поддерживаемым, рекомендуется **выносить маршруты в отдельные статические классы**.

  

### **Пример структуры каталогов**

```
MyApp/
 ├─ Endpoints/
 │   ├─ UserEndpoints.cs
 │   └─ ProductEndpoints.cs
 ├─ Program.cs
```

### **Реализация расширений для IEndpointRouteBuilder**

``` csharp
namespace MyApp.Endpoints
{
    public static class UserEndpoints
    {
        public static IEndpointRouteBuilder MapUserEndpoints(this IEndpointRouteBuilder app)
        {
            app.MapGet("/users", (IUserService userService) =>
            {
                return Results.Ok(userService.GetAllUsers());
            });

            app.MapGet("/users/{id:int}", (int id, IUserService userService) =>
            {
                var user = userService.GetUserById(id);
                return user is not null ? Results.Ok(user) : Results.NotFound();
            });

            return app;
        }
    }
}
```

### **Подключение эндпоинтов в** 

### **Program.cs**

``` csharp
var builder = WebApplication.CreateBuilder(args);
// Регистрация сервисов
builder.Services.AddScoped<IUserService, UserService>();

var app = builder.Build();

// Подключение эндпоинтов
app.MapUserEndpoints();

app.Run();
```

---

## **Рекомендации по структурированию Minimal API**

1. **Группируйте эндпоинты по доменам** — создавайте отдельный статический класс на каждый набор связанных маршрутов.
    
2. **Используйте расширения для IEndpointRouteBuilder** — это позволяет избежать громоздкого Program.cs.
    
3. **Применяйте Middleware для обработки ошибок и логирования**.
    
4. **Валидацию и бизнес-логику выносите в отдельные сервисы**.
    
5. **Используйте маршруты с префиксами** для логического разделения API (например, /api/users).
    

---

## **Итог**

  

Minimal API в ASP.NET Core — это мощный инструмент, который позволяет писать компактный, быстрый и легко поддерживаемый код. При правильной структуре и разделении ответственности он подходит даже для средних и больших проектов.
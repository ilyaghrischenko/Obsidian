## Что такое ASP.NET Core MVC?

ASP.NET Core MVC (Model-View-Controller) — это мощный и гибкий фреймворк от Microsoft для построения веб-приложений с четким разделением обязанностей между моделью, представлением и контроллером. Используется для построения как монолитных, так и модульных приложений.

---

## Архитектура MVC

- **Model (Модель)** — содержит бизнес-логику, сущности и работу с базой данных (например, через Entity Framework Core).
    
- **View (Представление)** — отвечает за отображение данных. Использует Razor-синтаксис в `.cshtml`файлах.
    
- **Controller (Контроллер)** — принимает запросы, взаимодействует с моделью, возвращает представление или другие результаты.
    

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        var model = new HomeViewModel { Message = "Welcome!" };
        return View(model);
    }
}
```

---

## Жизненный Цикл Запроса

1. Пользователь отправляет HTTP-запрос.
    
2. MVC определяет нужный контроллер и действие по маршруту.
    
3. Контроллер обрабатывает данные, взаимодействует с моделью.
    
4. Возвращается представление (HTML, JSON, файл и т.д.).
    

---

## Настройка в `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

---

## Views (`.cshtml` файлы)

### Основы Razor-синтаксиса

- `@model` — указывает тип модели для представления.
    
- `@{}` — блоки C#-кода.
    
- `@Html.DisplayFor(...)`, `@Html.EditorFor(...)` — хелперы отображения данных.
    

Пример `Views/Home/Index.cshtml`:

```html
@model HomeViewModel

<h1>@Model.Message</h1>
```

### Layout (`_Layout.cshtml`)

Обычно хранится в `Views/Shared/_Layout.cshtml`. Определяет общий шаблон страницы.

```html
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
</head>
<body>
    <div class="main">
        @RenderBody()
    </div>
</body>
</html>
```

Привязка layout:

```csharp
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
```

---

## Модели и ViewModel

Модели содержат бизнес-логику и структуры данных:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

ViewModel — облегчённая модель для передачи в представление:

```csharp
public class ProductViewModel
{
    public string Name { get; set; }
    public string DisplayPrice { get; set; }
}
```

---

## Валидация моделей

Поддерживается через атрибуты:

```csharp
public class RegisterViewModel
{
    [Required]
    public string Username { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
```

В представлении:

```html
<form asp-action="Register">
    <input asp-for="Username" />
    <span asp-validation-for="Username"></span>
    <button type="submit">Submit</button>
</form>
```

---

## Реальные кейсы использования MVC

1. **Сайты с регистрацией, аутентификацией, личным кабинетом.**
    
2. **Панели администратора с CRUD-операциями.**
    
3. **Многостраничные приложения с шаблонами.**
    
4. **Интеграция с БД через Entity Framework Core.**
    
5. **Локализация, отправка писем, загрузка файлов и т.п.**
    

---

## Частичные представления (`PartialView`)

Используются для переиспользуемых UI-компонентов:

```html
@await Html.PartialAsync("_ProductCard", product)
```

---

## Tag Helpers и HTML Helpers

- `asp-for`, `asp-action`, `asp-controller` упрощают генерацию HTML.
    

```html
<form asp-action="Login">
    <input asp-for="Username" />
    <button type="submit">Log In</button>
</form>
```

---

## Разделение на области (`Areas`)

Позволяют создавать модульную структуру:

```
Areas
 └─ Admin
    ├─ Controllers
    ├─ Views
    └─ Models
```

Регистрация маршрута:

```csharp
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

---

## Итог

ASP.NET Core MVC — зрелый фреймворк, идеально подходящий для приложений с разделением логики, UI и структуры. В реальных проектах активно используются Razor-представления, модели, валидация, layout-файлы и структура `Areas`.

Изучение ASP.NET Core MVC важно для создания поддерживаемых, расширяемых веб-приложений на платформе .NET.
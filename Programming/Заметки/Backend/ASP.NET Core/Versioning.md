## **Что такое версионирование API?**
  

Версионирование API — это механизм управления изменениями в вашем API без нарушения работы существующих клиентов. Оно позволяет поддерживать несколько версий одновременно, помечать устаревшие версии и постепенно переводить клиентов на новые.

---

## **Зачем нужно**

- **Обратная совместимость**: клиенты, использующие старые версии, продолжают работать.
    
- **Плавные обновления**: можно выпускать новые версии, не ломая старые.
    
- **Контроль жизненного цикла**: можно отмечать версии как устаревшие (Deprecated).
    

---

## **Установка пакета**

```
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

---

## **Настройка версионирования**

  

В Program.cs:

``` csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0); // версия по умолчанию 1.0
    options.AssumeDefaultVersionWhenUnspecified = true; // если версия не указана, брать дефолтную
    options.ReportApiVersions = true; // включить отчет о доступных версиях в заголовках
});
```

---

## **Версионирование для контроллеров**

  

### **Объявление версии:**

``` csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1() => Ok(new { Message = "Products V1" });

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2() => Ok(new { Message = "Products V2" });
}
```

### **Депрекейт версии:**

``` csharp
[ApiVersion("1.0", Deprecated = true)]
```

---

## **Версионирование для Minimal API**

  

### **Настройка:**

``` csharp
var versionSet = app.NewApiVersionSet()
    .HasApiVersion(new ApiVersion(1, 0))
    .HasApiVersion(new ApiVersion(2, 0))
    .ReportApiVersions()
    .Build();

app.MapGet("/products", () => new { Message = "Products V1" })
    .WithApiVersionSet(versionSet)
    .MapToApiVersion(new ApiVersion(1, 0));

app.MapGet("/products", () => new { Message = "Products V2" })
    .WithApiVersionSet(versionSet)
    .MapToApiVersion(new ApiVersion(2, 0));
```

### **Депрекейт версии:**

``` csharp
var versionSet = app.NewApiVersionSet()
    .HasApiVersion(new ApiVersion(1, 0))
    .HasDeprecatedApiVersion(new ApiVersion(1, 0))
    .HasApiVersion(new ApiVersion(2, 0))
    .ReportApiVersions()
    .Build();
```

---

## **Полезные фишки**

- **Версия по умолчанию**:
    

``` csharp
options.DefaultApiVersion = new ApiVersion(1, 0);
options.AssumeDefaultVersionWhenUnspecified = true;
```

-   
    
- **Множественные версии в одном контроллере**:
    

``` csharp
[ApiVersion("1.0")]
[ApiVersion("2.0")]
```

-   
    
- **Отчёт о версиях**:
    

``` csharp
options.ReportApiVersions = true;
```

- В ответе сервера будут заголовки:
    

``` csharp
api-supported-versions: 1.0, 2.0
api-deprecated-versions: 1.0
```

  

---

## **Когда использовать версионирование**

- При изменении контрактов API (новые поля, удаление старых).
    
- При больших изменениях логики, когда нельзя сохранить обратную совместимость.
    
- Для постепенного перевода клиентов на новые версии.
    

---

✅ Таким образом, ASP.NET Core поддерживает версионирование как для **контроллеров**, так и для **Minimal API** с удобным управлением версиями, включая депрекейт и отчёты.
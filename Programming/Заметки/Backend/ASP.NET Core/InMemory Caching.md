## **Введение**

In-Memory Caching — это способ хранения данных в оперативной памяти сервера для быстрого доступа. Он позволяет снизить нагрузку на базу данных и ускорить отклик API.

  

**Когда использовать:**

- Часто запрашиваемые данные
    
- Данные, редко изменяющиеся
    
- Кэширование результатов сложных вычислений
    

---

## **Подключение In-Memory Cache**

  

Чтобы использовать кэш, необходимо зарегистрировать его в контейнере зависимостей:

``` csharp
var builder = WebApplication.CreateBuilder(args);

// Регистрация In-Memory Cache
builder.Services.AddMemoryCache();

var app = builder.Build();

app.Run();
```

---

## **Пример использования в [[Minimal API]]**

``` csharp
app.MapGet("/products", async (IMemoryCache cache, IProductService productService) =>
{
    const string cacheKey = "products_cache";

    if (!cache.TryGetValue(cacheKey, out IEnumerable<ProductDto> products))
    {
        // Получение данных из сервиса/БД
        products = await productService.GetAllProductsAsync();

        // Настройки кэша
        var cacheOptions = new MemoryCacheEntryOptions()
            .SetAbsoluteExpiration(TimeSpan.FromMinutes(5)) // Время жизни кэша
            .SetSlidingExpiration(TimeSpan.FromMinutes(2)); // Продление при обращении

        cache.Set(cacheKey, products, cacheOptions);
    }

    return Results.Ok(products);
});
```

---

## **Использование** 

## **GetOrCreateAsync**

  

Метод GetOrCreateAsync упрощает работу с кэшем — он автоматически проверяет наличие данных по ключу и, если их нет, создаёт новый элемент.

``` csharp
app.MapGet("/categories", async (IMemoryCache cache, ICategoryService categoryService) =>
{
    var categories = await cache.GetOrCreateAsync("categories_cache", async entry =>
    {
        entry.SetAbsoluteExpiration(TimeSpan.FromMinutes(10));
        entry.SetSlidingExpiration(TimeSpan.FromMinutes(3));

        return await categoryService.GetAllCategoriesAsync();
    });

    return Results.Ok(categories);
});
```

**Преимущество:** код становится компактнее, а логика получения и сохранения в кэш — централизованной.

---

## **Настройки кэширования**

  

### **Absolute Expiration (абсолютное время жизни)**

  

Элемент удаляется из кэша через заданное время, независимо от обращений.

``` csharp
.SetAbsoluteExpiration(TimeSpan.FromMinutes(5))
```

### **Sliding Expiration (скользящее время жизни)**

  

Время жизни продлевается при каждом обращении к элементу.

``` csharp
.SetSlidingExpiration(TimeSpan.FromMinutes(2))
```

### **Приоритет удаления (CacheItemPriority)**

``` csharp
.SetPriority(CacheItemPriority.High)
```

---

## **Очистка кэша**

  

Чтобы удалить данные из кэша:

``` csharp
cache.Remove("products_cache");
```

---

## **Рекомендации**

1. **Не храните слишком большие объёмы данных** — это может привести к нехватке памяти.
    
2. **Используйте понятные ключи** для кэшируемых объектов.
    
3. **Комбинируйте с Distributed Cache**, если приложение запущено в нескольких инстансах.
    
4. **Следите за временем жизни кэша**, чтобы не отдавать устаревшие данные.
    
5. **Логируйте кэш-хиты и промахи** для оптимизации.
    

---

## **Итог**

  

In-Memory Caching в ASP.NET Core — это простой и эффективный способ ускорить работу API. Он особенно полезен для небольших и средних проектов или при кэшировании данных, которые часто используются и редко меняются. Использование GetOrCreateAsync позволяет писать более компактный и читаемый код при работе с кэшем.
tags: [#backend, #database, #entity-framework-core, #bulk-operations]

## **Что это такое**


Bulk-операции в **Entity Framework Core 7+** позволяют выполнять массовые изменения или удаления записей напрямую в базе данных **без предварительной загрузки сущностей в память**.
  

До появления этих методов для изменения/удаления большого числа строк приходилось:

1. Загружать все сущности в память.
    
2. Изменять их.
    
3. Сохранять через SaveChanges().
    


Это было неэффективно. Теперь EF Core генерирует SQL напрямую.


## **Основные методы**

### **1. ExecuteUpdate**

Позволяет обновить записи в таблице, соответствующие LINQ-запросу.

``` csharp
await context.Customers
    .Where(c => c.IsActive == false)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(c => c.IsActive, true)
        .SetProperty(c => c.UpdatedDate, DateTime.UtcNow));
```

👉 SQL, который сгенерирует EF Core:

``` csharp
UPDATE [Customers]
SET [IsActive] = 1, [UpdatedDate] = GETUTCDATE()
WHERE [IsActive] = 0;
```

- SetProperty — указывает, какое свойство обновить.
    
- Можно цепочкой задавать несколько свойств.
    
- Работает только с **простыми выражениями**, нельзя подставлять сложную бизнес-логику.
    

  

### **2. ExecuteDelete**

Позволяет удалить записи напрямую.

``` csharp
await context.Orders
    .Where(o => o.Status == OrderStatus.Cancelled)
    .ExecuteDeleteAsync();
```

👉 SQL:

``` sql
DELETE FROM [Orders]
WHERE [Status] = 'Cancelled';
```

- Удаляет строки **без загрузки в память**.
    
- Не вызывает **события жизненного цикла EF** (например, OnDelete, SaveChanges, ChangeTracker).
    

## **Когда использовать**

✅ Использовать bulk-операции стоит, если нужно:

- Массово обновить или удалить строки.
    
- Избежать загрузки большого числа объектов в память.
    
- Ускорить выполнение запросов.
    

❌ Не использовать, если:

- Нужны **события EF (hooks, triggers в коде)**.
    
- Требуется сложная бизнес-логика при изменении.
    

## **Ограничения**

- **Change Tracker** не знает об изменениях → если в памяти загружены сущности, их состояние **будет устаревшим**.
    
- Не поддерживает **навигационные свойства**.
    
- Работает только с **простыми выражениями** (например, c => true, o => o.Price * 0.9).
    

  

## **Пример: массовое повышение цен**

``` csharp
await context.Products
    .Where(p => p.Category == "Electronics")
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(p => p.Price, p => p.Price * 1.1));
```

👉 SQL:

``` sql
UPDATE [Products]
SET [Price] = [Price] * 1.1
WHERE [Category] = 'Electronics';
```

## **Сравнение с классическим подходом**

  

### **Старый способ (дорого):**

``` csharp
var customers = await context.Customers.Where(c => !c.IsActive).ToListAsync();
foreach (var customer in customers)
{
    customer.IsActive = true;
    customer.UpdatedDate = DateTime.UtcNow;
}
await context.SaveChangesAsync();
```

### **Новый способ (эффективно):**

``` csharp
await context.Customers
    .Where(c => !c.IsActive)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(c => c.IsActive, true)
        .SetProperty(c => c.UpdatedDate, DateTime.UtcNow));
```

## **Вывод**

- ExecuteUpdate и ExecuteDelete — инструменты для **массовых операций напрямую в БД**.
    
- Они **не используют ChangeTracker**, поэтому быстрые, но не триггерят EF-хуки.
    
- Отлично подходят для batch-обновлений и batch-удалений.
    
- Нужно помнить о консистентности между базой и объектами в памяти.
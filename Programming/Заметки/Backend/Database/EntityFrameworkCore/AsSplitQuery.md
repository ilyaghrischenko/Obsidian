tags: [#backend, #database, #entity-framework-core, #assplitquery]

## **Что это такое**

AsSplitQuery — метод в **Entity Framework Core**, позволяющий управлять стратегией загрузки связанных данных (Include).

По умолчанию EF Core старается выполнить один SQL-запрос (Single Query) с JOIN для загрузки всех связанных данных. Но это может привести к:

- **Картезианскому взрыву** (множество дублирующихся строк при сложных Include).
    
- Высокой нагрузке на сеть и БД.
    
- Сложности оптимизации.
    

AsSplitQuery решает эту проблему, разделяя запрос на несколько SQL-запросов.

  

## **Зачем нужно**

✅ Преимущества AsSplitQuery:

- Уменьшает количество дублирующихся строк.
    
- Делает запросы проще и быстрее в некоторых сценариях.
    
- Удобно при загрузке данных с множеством Include.
    

❌ Недостатки:

- Генерирует несколько запросов вместо одного.
    
- Может быть дороже, если сетевые задержки критичны.
    

## **Как использовать**


### **1. Пример без AsSplitQuery(Single Query)**

``` csharp
var orders = await context.Orders
    .Include(o => o.Customer)
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
    .ToListAsync();
```

👉 SQL будет содержать JOIN на все связанные таблицы. Если у заказа 10 товаров, данные заказа и покупателя будут дублироваться 10 раз.

  
### **2. Пример с AsSplitQuery**

``` csharp
var orders = await context.Orders
    .Include(o => o.Customer)
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
    .AsSplitQuery()
    .ToListAsync();
```

👉 EF Core разобьёт запрос на несколько:

``` sql
SELECT * FROM Orders;
SELECT * FROM Customers WHERE Id IN (...);
SELECT * FROM OrderItems WHERE OrderId IN (...);
SELECT * FROM Products WHERE Id IN (...);
```

### **Глобальная настройка**


Можно задать стратегию по умолчанию:

``` csharp
optionsBuilder.UseSqlServer(connectionString, 
    o => o.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery));
```

Или через контекст:

``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery);
}
```

## **Когда применять**

Используйте AsSplitQuery, если:

- У вас много Include.
    
- Есть риск картезианского взрыва.
    
- Нужно оптимизировать нагрузку на БД.
    

Не используйте, если:

- Запросы должны выполняться строго одним SQL.
    
- Важна минимизация количества SQL-запросов.
    

## **Сравнение**


### **1. Single Query (без AsSplitQuery)**

- Один SQL-запрос.
    
- Возможны дубликаты и тяжёлый JOIN.
    

### **2. Split Query (с AsSplitQuery)**

- Несколько SQL-запросов.
    
- Нет картезианского взрыва, но больше round-trip’ов к БД.
    

## **Вывод**

- AsSplitQuery — инструмент оптимизации запросов с Include.
    
- Полезен при сложных связях и больших данных.
    
- Нужно выбирать стратегию в зависимости от сценария: SingleQuery (меньше запросов) или SplitQuery (меньше дубликатов).
    
- Гибкость: можно указывать локально (.AsSplitQuery()) или глобально (через UseQuerySplittingBehavior).
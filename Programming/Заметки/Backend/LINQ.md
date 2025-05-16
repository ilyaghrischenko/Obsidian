
LINQ (Language Integrated Query) — это мощная технология в .NET, которая позволяет писать запросы к различным источникам данных (коллекциям в памяти, базам данных, XML, файлам и т.д.) прямо в C# с помощью декларативного синтаксиса.

---

## Преимущества LINQ

- Унифицированный подход к запросам
    
- Поддержка IntelliSense и компиляции
    
- Безопасность типов
    
- Более читаемый и краткий код
    

---

## Основные пространства имён

```csharp
using System.Linq; // Для LINQ to Objects
using System.Xml.Linq; // Для LINQ to XML
using Microsoft.EntityFrameworkCore; // Для LINQ to Entities (EF Core)
```

---

## Источники данных для LINQ

- **LINQ to Objects**: запросы к коллекциям в памяти (`List`, `Array`, `Dictionary` и т.д.)
    
- **LINQ to SQL/Entities**: запросы к базам данных через ORM
    
- **LINQ to XML**: работа с XML-документами
    
- **LINQ to JSON**: через внешние библиотеки (например, Newtonsoft.Json)
    

---

## Синтаксис LINQ

### 1. Синтаксис запросов (Query Syntax)

```csharp
var result = from n in numbers
             where n % 2 == 0
             select n;
```

### 2. Метод-синтаксис (Method Syntax / Fluent API)

```csharp
var result = numbers.Where(n => n % 2 == 0).Select(n => n);
```

Оба подхода дают одинаковый результат.

---

## Основные методы LINQ

### Фильтрация

- `Where(predicate)` — фильтрация по условию
    

### Проекция

- `Select(selector)` — преобразование элементов
    
- `SelectMany(selector)` — "плоская" проекция (например, раскрытие списков в списке)
    

### Сортировка

- `OrderBy(keySelector)` / `OrderByDescending(...)`
    
- `ThenBy(...)` / `ThenByDescending(...)`
    

### Агрегация

- `Count()`, `Sum()`, `Average()`, `Min()`, `Max()`
    
- `Aggregate()` — пользовательская агрегация
    

### Проверка условий

- `Any(predicate)` — есть ли хоть один
    
- `All(predicate)` — все ли соответствуют
    
- `Contains(value)` — содержит ли элемент
    

### Элементы

- `First()`, `FirstOrDefault()`
    
- `Single()`, `SingleOrDefault()`
    
- `Last()`, `LastOrDefault()`
    
- `ElementAt(index)`
    

### Объединение и пересечение

- `Concat()`, `Union()`, `Intersect()`, `Except()`
    

### Группировка

- `GroupBy(keySelector)`
    

### Объединение (Join)

- `Join(...)`, `GroupJoin(...)`
    

---

## Примеры

### Фильтрация чётных чисел

```csharp
var evens = numbers.Where(n => n % 2 == 0);
```

### Выбор имён пользователей

```csharp
var names = users.Select(u => u.Name);
```

### Группировка по возрасту

```csharp
var grouped = users.GroupBy(u => u.Age);
```

### Объединение двух коллекций по ID

```csharp
var result = from u in users
             join o in orders on u.Id equals o.UserId
             select new { u.Name, o.Product };
```

---

## Отличие от SQL

- LINQ работает с объектами
    
- Результаты можно легко использовать в коде
    
- Выполняется безопасно и предсказуемо, особенно с in-memory коллекциями
    

---

## Deferred vs Immediate Execution

- **Отложенное выполнение (Deferred)**: `Where`, `Select`, `OrderBy` — вычисляются при перечислении
    
- **Немедленное выполнение (Immediate)**: `ToList`, `Count`, `Sum` — выполняются сразу
    

```csharp
var query = numbers.Where(n => n > 5); // не выполнится сразу
var result = query.ToList(); // выполнение произойдёт здесь
```

---

## Советы по использованию

- Используйте `ToList()` или `ToArray()` для немедленного выполнения при необходимости
    
- Не вызывайте `ToList()` до фильтрации — это замедлит производительность
    
- Используйте `FirstOrDefault()` вместо `First()`, если коллекция может быть пустой
    

---

## Дополнительные ресурсы

- [LINQPad](https://www.linqpad.net/) — среда для экспериментов с LINQ
    
- Microsoft Docs: [LINQ](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)
    

---

## Заключение

LINQ — это мощный инструмент, который позволяет писать более чистый, декларативный и выразительный код при работе с данными. Он применим во многих частях .NET разработки: от фильтрации коллекций до составления сложных SQL-запросов через Entity Framework.
## 1. Что это и зачем?

Внедрение зависимостей (Dependency Injection, DI) — это способ управления зависимостями в приложении. Это позволяет:

- Упростить код и сделать его более гибким.
    
- Улучшить тестируемость (можно подменять зависимости в тестах).
    
- Снизить связанность компонентов.
    

В ASP.NET Core три основных вида жизненного цикла зависимостей:

- **Transient** (краткоживущие)
    
- **Scoped** (живут в пределах запроса)
    
- **Singleton** (один экземпляр на всё приложение)
    

---

## 2. Примеры

### **Transient** – создаётся новый объект при каждом запросе

Используется для легковесных сервисов, которые не хранят состояние.

``` csharp
public interface IMessageService
{
    void SendMessage(string message);
}

public class TransientMessageService : IMessageService
{
    private readonly Guid _id = Guid.NewGuid();
    public void SendMessage(string message)
    {
        Console.WriteLine($"[{_id}] Сообщение: {message}");
    }
}
```

Регистрация в `Program.cs`:

``` csharp
builder.Services.AddTransient<IMessageService, TransientMessageService>();
```

Использование:

``` csharp
var service1 = scope.ServiceProvider.GetRequiredService<IMessageService>();
var service2 = scope.ServiceProvider.GetRequiredService<IMessageService>();

service1.SendMessage("Первый вызов");
service2.SendMessage("Второй вызов");
```

При каждом получении зависимости создаётся новый объект. Выведенные `Guid` будут разными.

---

### **Scoped** – один экземпляр на HTTP-запрос

Используется для сервисов, которые должны быть общими в пределах одного запроса, например, работа с базой данных.

``` csharp
public class ScopedMessageService : IMessageService
{
    private readonly Guid _id = Guid.NewGuid();
    public void SendMessage(string message)
    {
        Console.WriteLine($"[{_id}] Сообщение: {message}");
    }
}
```

Регистрация в `Program.cs`:

``` csharp
builder.Services.AddScoped<IMessageService, ScopedMessageService>();
```

Использование:

``` csharp
using (var scope1 = app.Services.CreateScope())
{
    var service1 = scope1.ServiceProvider.GetRequiredService<IMessageService>();
    var service2 = scope1.ServiceProvider.GetRequiredService<IMessageService>();
    
    service1.SendMessage("Первый вызов");
    service2.SendMessage("Второй вызов");
}

using (var scope2 = app.Services.CreateScope())
{
    var service3 = scope2.ServiceProvider.GetRequiredService<IMessageService>();
    service3.SendMessage("Третий вызов");
}
```

В пределах одного `scope` `service1` и `service2` будут иметь одинаковый `Guid`, но `service3` получит новый `Guid`.

---

### **Singleton** – один экземпляр на всё приложение

Используется для сервисов, которые должны существовать в единственном экземпляре, например, кэширование или конфигурации.

``` csharp
public class SingletonMessageService : IMessageService
{
    private readonly Guid _id = Guid.NewGuid();
    public void SendMessage(string message)
    {
        Console.WriteLine($"[{_id}] Сообщение: {message}");
    }
}
```

Регистрация в `Program.cs`:

``` csharp
builder.Services.AddSingleton<IMessageService, SingletonMessageService>();
```

Использование:

``` csharp
var service1 = app.Services.GetRequiredService<IMessageService>();
var service2 = app.Services.GetRequiredService<IMessageService>();

service1.SendMessage("Первый вызов");
service2.SendMessage("Второй вызов");
```

Оба объекта будут иметь одинаковый `Guid`, так как создаётся один экземпляр на всё время работы приложения.

---

## 3. Когда использовать?

| Тип       | Когда использовать?                                                                |
| --------- | ---------------------------------------------------------------------------------- |
| Transient | Лёгкие, независимые сервисы (например, логирование в консоль)                      |
| Scoped    | Рабочие сервисы в рамках одного HTTP-запроса (например, репозитории, Unit of Work) |
| Singleton | Глобальные сервисы (например, кэширование, конфигурация, глобальные настройки)     |

Выбор типа зависит от логики приложения, но **Scoped** чаще всего используется в веб-приложениях.
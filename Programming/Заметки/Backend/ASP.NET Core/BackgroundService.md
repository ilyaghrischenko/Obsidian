## **1. Что такое** 

## **BackgroundService**

  

BackgroundService — это абстрактный базовый класс, предоставляемый ASP.NET Core, который упрощает реализацию фоновых задач. Он реализует интерфейс IHostedService и предоставляет защищённый метод ExecuteAsync, в котором размещается ваша фоновая логика.

  

Он предназначен для задач, которые должны выполняться асинхронно и непрерывно в течение всего времени работы приложения.

---

## **2. Наследование и структура**

  

Класс BackgroundService уже реализует StartAsync и StopAsync, а вы реализуете только один метод:

``` csharp
protected abstract Task ExecuteAsync(CancellationToken stoppingToken);
```

ASP.NET Core сам заботится о передаче токена отмены, когда приложение завершает работу.

---

## **3. Пример использования**

``` csharp
public class MyBackgroundWorker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine($"[{DateTime.Now}] Фоновая задача выполняется...");
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }

        Console.WriteLine("Фоновая задача остановлена.");
    }
}
```

### **Регистрация в** 

### **Program.cs**

###  **или** 

### **Startup.cs**

### **:**

``` csharp
builder.Services.AddHostedService<MyBackgroundWorker>();
```

## **4. Обработка исключений**

  

Важно оборачивать код в try-catch, чтобы избежать падения службы:

``` csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    try
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // Ваша логика
            await Task.Delay(1000, stoppingToken);
        }
    }
    catch (Exception ex)
    {
        // Логирование ошибки
        Console.WriteLine($"Ошибка: {ex.Message}");
    }
}
```

## **5. Использование зависимостей**

  

Если необходимо использовать другие сервисы, внедряйте их через конструктор:

``` csharp
public class EmailSenderService : BackgroundService
{
    private readonly IEmailSender _emailSender;

    public EmailSenderService(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await _emailSender.SendPendingEmailsAsync();
            await Task.Delay(60000, stoppingToken); // раз в минуту
        }
    }
}
```

## **6. Практические кейсы**

- Обработка очередей (RabbitMQ, Kafka, Redis и т.д.).
    
- Периодическое выполнение задач (таймеры, cron-подобные задачи).
    
- Слежение за внешними источниками (например, API или БД).
    
- Уведомления, логирование, сбор метрик и т.д.
    

---

## **7. Полезные советы**

  

✅ Используйте CancellationToken корректно для остановки задач.

✅ Обрабатывайте исключения, иначе служба завершится.

✅ Не блокируйте поток (Thread.Sleep), используйте await Task.Delay.

✅ Разносите бизнес-логику по отдельным сервисам.

✅ Используйте ILogger для логирования.

---

## **8. Отличие от** 

## **[[IHostedService]]**

| **Характеристика**          | IHostedService         | BackgroundService                            |
| --------------------------- | ---------------------- | -------------------------------------------- |
| Реализация                  | Интерфейс              | Абстрактный класс                            |
| Метод фоновой логики        | StartAsync / StopAsync | ExecuteAsync                                 |
| Простота использования      | Более низкоуровневый   | Более удобный для непрерывной фоновой логики |
| Поддержка CancellationToken | Вручную                | Встроено в ExecuteAsync                      |

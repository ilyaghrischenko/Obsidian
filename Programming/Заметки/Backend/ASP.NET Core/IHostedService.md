## **1. Что такое** 

## **IHostedService**

  

IHostedService — это интерфейс, предоставляемый ASP.NET Core, который позволяет запускать фоновые службы в рамках жизненного цикла приложения. Такие службы запускаются при старте приложения и могут выполнять задачи в фоне, пока приложение работает.

  

Он определяет два метода:

``` csharp
Task StartAsync(CancellationToken cancellationToken);
Task StopAsync(CancellationToken cancellationToken);
```

## **2. Зачем нужен** 

## **IHostedService**

  

Он используется, когда нужно:

- Выполнять периодические задачи (например, очистку логов, синхронизацию с внешними сервисами и т.д.).
    
- Поддерживать постоянные соединения (например, с очередями RabbitMQ, Kafka и др.).
    
- Запускать службы, не зависящие от запросов пользователя.
    
- Реализовать таймеры, планировщики задач.

## **3. Как использовать**

  

### **Пример реализации фоновой службы:**

``` csharp
public class MyBackgroundService : IHostedService, IDisposable
{
    private Timer _timer;

    public Task StartAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Служба запущена.");
        _timer = new Timer(DoWork, null, TimeSpan.Zero, TimeSpan.FromSeconds(5));
        return Task.CompletedTask;
    }

    private void DoWork(object state)
    {
        Console.WriteLine("Выполнение фоновой задачи: " + DateTime.Now);
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Служба остановлена.");
        _timer?.Change(Timeout.Infinite, 0);
        return Task.CompletedTask;
    }

    public void Dispose()
    {
        _timer?.Dispose();
    }
}
```

### **Регистрация в DI контейнере:**

``` csharp
builder.Services.AddHostedService<MyBackgroundService>();
```

## **4. Использование** 

## **BackgroundService**

  

Для упрощения реализации можно унаследоваться от абстрактного класса BackgroundService:

``` csharp
public class TimedHostedService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Работа в фоне: " + DateTime.Now);
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }
    }
}
```

Регистрация происходит аналогично:

``` csharp
builder.Services.AddHostedService<TimedHostedService>();
```

## **5. Жизненный цикл**

- StartAsync вызывается при запуске приложения.
    
- ExecuteAsync (если используется BackgroundService) работает в фоне до тех пор, пока приложение не завершится.
    
- StopAsync вызывается при остановке приложения.
    
- Если реализуется IDisposable, будет вызван Dispose.
    

---

## **6. Практические кейсы**

- Обновление кеша из БД.
    
- Отправка e-mail уведомлений.
    
- Обработка задач из очереди.
    
- Мониторинг состояния системы.
    

---

## **7. Полезные советы**

- Используйте CancellationToken для корректной остановки.
    
- Не блокируйте поток (избегайте Thread.Sleep, используйте Task.Delay).
    
- Если логика сложная, лучше разделить её на отдельные сервисы и внедрить через DI.
tags: [#backend, #cancellation-token]

## **🧾 Что такое** 

## **CancellationToken**

  

CancellationToken — это встроенный механизм .NET, позволяющий **отменять асинхронные операции**. Он используется для того, чтобы по команде остановить выполнение кода, не выбрасывая исключений вручную и не завершая процесс насильно.

  

Полезен при:

- Завершении долгих операций
    
- Завершении задач при закрытии приложения
    
- Обработке отмены HTTP-запросов, фоновых задач и т.д.
    

---

## **🧠 Основные участники**
|**Компонент**|**Описание**|
|---|---|
|CancellationToken|Токен, который передаётся в метод и сигнализирует об отмене|
|CancellationTokenSource|Источник, управляющий токеном (можно вызвать .Cancel())|
|IsCancellationRequested|Проверка, была ли запрошена отмена|

---

## **🔧 Пример использования**

``` csharp
public async Task DownloadDataAsync(CancellationToken token)
{
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested(); // выбрасывает исключение, если отмена запрошена

        Console.WriteLine($"Загрузка {i + 1}/10...");
        await Task.Delay(1000, token); // важно передавать токен
    }

    Console.WriteLine("Загрузка завершена");
}
```

### **Пример вызова с отменой:**

``` csharp
var cts = new CancellationTokenSource();

var task = DownloadDataAsync(cts.Token);

// Отменить через 3 секунды
Task.Delay(3000).ContinueWith(_ => cts.Cancel());

await task;
```

## **❗ Важные методы и свойства**
|**Метод / Свойство**|**Назначение**|
|---|---|
|token.IsCancellationRequested|Проверяет, запрошена ли отмена|
|token.ThrowIfCancellationRequested()|Выбрасывает OperationCanceledException|
|token.Register(Action)|Вызывает действие при отмене|
|cts.Cancel()|Запрашивает отмену|
|cts.CancelAfter(ms)|Автоматическая отмена через время|
|using var cts = new CancellationTokenSource()|Правильная очистка ресурсов|

## **🧬 Совместимость с** 

## **Task**

``` csharp
await Task.Delay(5000, token); // отменяется, если токен сработал
```

## **📌 Использование в ASP.NET Core**

  

Во многих встроенных точках ASP.NET Core, таких как Controller, [[FastEndpoints]], Minimal API, [[BackgroundService]], [[CQRS + Mediator]], есть автоматическая поддержка CancellationToken:

``` csharp
[HttpGet]
public async Task<IActionResult> GetData(CancellationToken cancellationToken)
{
    var data = await _dataService.LoadAsync(cancellationToken);
    return Ok(data);
}
```

## **🔄 Пример в BackgroundService**

``` csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        // Выполняем задачу
        await DoWorkAsync(stoppingToken);
    }
}
```

---

## **🛑 Хорошие практики**

  

✅ Используйте ThrowIfCancellationRequested() там, где нужно быстро прерывать выполнение

✅ Всегда передавайте CancellationToken в поддерживаемые методы (Task.Delay, HttpClient, [[Entity Framework Core]] и т.д.)

✅ В фоновом сервисе или долгих обработках никогда не игнорируйте токен

✅ Избегайте Thread.Sleep, используйте Task.Delay(..., token)

---

## **🧪 Итог**

  

CancellationToken — это мощный инструмент для написания **гибкого, отзывчивого и управляемого** асинхронного кода. Он помогает:

- избегать подвисаний
    
- корректно завершать задачи
    
- эффективно использовать ресурсы
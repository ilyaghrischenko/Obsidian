tags: [#azure, #azure-functions]

## Что такое Azure Functions?

Azure Functions — это бессерверная платформа от Microsoft для выполнения кода в ответ на события. Вы пишете только код, а инфраструктура масштабируется автоматически. Это идеальный инструмент для обработки фоновых задач, интеграции между сервисами и автоматизации процессов.

---

## Основные особенности Azure Functions

- **Serverless**: не нужно управлять серверами.
    
- **Событийно-ориентированная модель**: функция запускается по триггеру.
    
- **Масштабируемость**: автоматически масштабируется в зависимости от нагрузки.
    
- **Поддержка C#, JavaScript, Python, Java, PowerShell и других языков**.
    
- **Интеграция с другими сервисами Azure**.
    

---

## Виды триггеров (Triggers)

Триггер — это событие, которое вызывает выполнение функции.  
Ниже приведены **основные и часто используемые триггеры**:

### 1. **HTTP Trigger** ✅

Запускает функцию при HTTP-запросе (GET, POST и т.д.).

Пример:

```csharp
[Function("HttpExample")]
public HttpResponseData Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData req)
{
    var response = req.CreateResponse(HttpStatusCode.OK);
    response.WriteString("Hello from HTTP trigger");
    return response;
}
```

📌 Используется для создания API, webhook-обработчиков и внешнего взаимодействия.

---

### 2. **Timer Trigger** ✅

Запускает функцию по расписанию (cron-выражение).

Пример:

```csharp
[Function("TimerExample")]
public void Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, FunctionContext context)
{
    logger.LogInformation($"Функция сработала: {DateTime.Now}");
}
```

📌 Подходит для фоновых задач: очистка, бэкапы, синхронизация.

---

### 3. **Queue Trigger (Azure Storage Queue) [[Очереди (Azure Queue, RabbitMQ, etc...)]]** ✅

Запускает функцию при поступлении сообщения в очередь.

Пример:

```csharp
[Function("QueueFunction")]
public void Run([QueueTrigger("myqueue", Connection = "AzureWebJobsStorage")] string message)
{
    logger.LogInformation($"Сообщение из очереди: {message}");
}
```

📌 Идеален для фоновой обработки задач и разделения ответственности между сервисами.

---

### 4. **Blob Trigger**

Запускает функцию при загрузке/изменении файла в [[Blob Storage]].

Пример:

```csharp
[Function("BlobTriggerExample")]
public void Run([BlobTrigger("images/{name}", Connection = "AzureWebJobsStorage")] Stream blob, string name)
{
    logger.LogInformation($"Новый blob: {name}");
}
```

📌 Полезен для обработки изображений, видео, логов и других файлов.

---

### 5. **Service Bus Trigger**

Запускает функцию при поступлении сообщения в Azure Service Bus (Queue или Topic).

Пример:

```csharp
[Function("ServiceBusExample")]
public void Run([ServiceBusTrigger("queue-name", Connection = "ServiceBusConnection")] string message)
{
    logger.LogInformation($"Сообщение из Service Bus: {message}");
}
```

📌 Используется в распределённых системах для более надёжной доставки сообщений.

---

### Дополнительные триггеры

- **Cosmos DB Trigger** — запускается при изменениях в Cosmos DB.
    
- **Event Grid Trigger** — реагирует на события из других сервисов Azure.
    
- **Event Hub Trigger** — используется для обработки телеметрии и событий в реальном времени.
    
- **SignalR Trigger** — (реже используется) поддерживает реальное время.
    

---

## Важные концепции и возможности

### Bindings (Привязки)

Позволяют легко подключаться к другим ресурсам без ручного написания кода для подключения:

- Input binding: получать данные
    
- Output binding: отправлять данные
    

Пример Output Binding:

```csharp
[Function("QueueOutputExample")]
[QueueOutput("outqueue")]
public string Run([HttpTrigger(AuthorizationLevel.Anonymous, "get")] HttpRequestData req)
{
    return "Сообщение для другой очереди";
}
```

---

### Разработка и деплой

- **Инструменты**: Visual Studio, VS Code, Azure CLI, GitHub Actions.
    
- **Режимы хостинга**:
    
    - Consumption (автомасштабируемый, платишь за вызовы)
        
    - Premium (больше возможностей, кастомные масштабы)
        
    - Dedicated (на App Service Plan)
        

---

### Конфигурация

- `local.settings.json` для разработки локально.
    
- Переменные окружения (`EnvironmentVariables`) для конфиденциальной информации (ключи, строки подключения).
    

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet"
  }
}
```

---

## Когда использовать Azure Functions

✅ Подходит:

- Для автоматизации (cron, email, отчёты)
    
- Для фоновых задач и микросервисов
    
- Для webhook-обработчиков
    
- Для IoT/обработки потоков событий
    

🚫 Не подходит:

- Для долгих задач > 5-10 мин (используй Durable Functions)
    
- Для задач, требующих постоянного состояния
    

---

## Заключение

Azure Functions — мощный инструмент для построения современных, масштабируемых, событийно-ориентированных приложений. Понимание основных триггеров, привязок и сценариев использования позволяет создавать эффективные решения с минимальным кодом и затратами.
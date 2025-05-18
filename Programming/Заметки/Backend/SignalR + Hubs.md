## Что такое SignalR?

**SignalR** — это библиотека от Microsoft для ASP.NET Core, которая упрощает процесс добавления **реального времени (real-time)** в веб-приложения. SignalR позволяет серверному коду отправлять сообщения сразу всем подключённым клиентам, отдельным клиентам или группам клиентов.

Используется для:

- Уведомлений (например, "уведомление о новом сообщении")
    
- Чатов
    
- Онлайн-игр
    
- Панелей мониторинга (dashboard)
    
- Любых приложений, где важна моментальная реакция клиента
    

---

## Основные понятия

### Hub

**Hub** — это центральная точка, через которую сервер и клиент обмениваются сообщениями. Вы создаёте свой класс Hub, а SignalR сам управляет подключениями.

```csharp
public class NotificationHub : Hub<INotificationHubClient>
{
    // Сервер вызывает этот метод на клиентах
    public async Task SendNotification(string message)
    {
        await Clients.All.SendNotification(message);
    }
}
```

### INotificationHubClient

Это интерфейс, описывающий методы, которые сервер может вызывать на клиенте.

```csharp
public interface INotificationHubClient
{
    Task SendNotification(string message);
}
```

---

## Установка и настройка

### Установка NuGet пакета

```sh
dotnet add package Microsoft.AspNetCore.SignalR
```

### Добавление SignalR в сервер

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSignalR();

var app = builder.Build();
app.MapHub<NotificationHub>("/notificationHub");
```

---

## Клиентская часть (JavaScript)

```html
<script src="https://cdn.jsdelivr.net/npm/@microsoft/signalr@latest/dist/browser/signalr.min.js"></script>
<script>
    const connection = new signalR.HubConnectionBuilder()
        .withUrl("/notificationHub")
        .build();

    connection.on("SendNotification", (message) => {
        console.log("Notification from server:", message);
    });

    connection.start();
</script>
```

---

## Вызов клиента с сервера

```csharp
await _hubContext.Clients.All.SendNotification("Отчёт готов!");
```

Можно также:

- `Clients.Caller` — отправить сообщение только текущему клиенту
    
- `Clients.Others` — всем, кроме текущего
    
- `Clients.User(userId)` — конкретному пользователю
    
- `Clients.Group(groupName)` — определённой группе
    

---

## SignalR и Azure

В Azure вы можете использовать **Azure SignalR Service**, если не хотите держать постоянные подключения на своём сервере.

### Почему?

- Масштабируемость (тысячи подключений)
    
- Упрощённая интеграция с Azure Functions
    
- Меньше нагрузки на сервер
    

---

## SignalR с Azure Functions

В Azure Functions нельзя использовать обычный Hub как в ASP.NET, но можно взаимодействовать через **Azure SignalR Service** и использовать **SignalR output bindings**:

### Пример:

```csharp
[Function("NotifyClients")]
public static async Task Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    [SignalR(HubName = "notifications")] IAsyncCollector<SignalRMessage> signalRMessages)
{
    await signalRMessages.AddAsync(new SignalRMessage
    {
        Target = "SendNotification",
        Arguments = ["Привет из Azure Functions!"]
    });
}
```

⚠️ Для этого нужно создать Azure SignalR Service в портале Azure.

---

## Совмещение с Azure Queue + SignalR

1. **Клиент** отправляет данные через Web API.
    
2. **Сервер** кладёт сообщение в очередь (Azure Queue).
    
3. **Azure Function** реагирует на сообщение из очереди, выполняет задачу (например, создаёт отчёт).
    
4. **Azure Function** отправляет уведомление клиентам через SignalR (через Azure SignalR Service или Web API).
    

---

## Итог

SignalR — мощный инструмент для real-time приложений. Он позволяет легко настроить мгновенные уведомления и взаимодействие между сервером и клиентами. Чаще всего используется в связке с:

- **ASP.NET Core API/Web** для маршрутов и бизнес-логики
    
- **Azure Queue** для отложенных/фоных задач
    
- **Azure Functions** для независимых обработчиков
    
- **SignalR** для доставки информации клиенту
    

> Такая архитектура хорошо масштабируется, снижает нагрузку на основной сервер, и позволяет реагировать на события в реальном времени.
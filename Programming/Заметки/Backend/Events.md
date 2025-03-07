## 1. Что такое события?

События в C# позволяют объектам уведомлять другие объекты о произошедших изменениях. Они основаны на механизме [[Delegates]] и позволяют реализовать шаблон "издатель-подписчик".

## 2. Объявление события

Событие объявляется в классе с использованием ключевого слова `event` и делегата:

```csharp
public class Publisher
{
    public event Action MyEvent;
}
```

## 3. Подписка и отписка от события

Подписка на событие выполняется с помощью оператора `+=`, а отписка – `-=`:

```csharp
public class Subscriber
{
    public void OnEventTriggered()
    {
        Console.WriteLine("Событие произошло!");
    }
}

Publisher publisher = new Publisher();
Subscriber subscriber = new Subscriber();

publisher.MyEvent += subscriber.OnEventTriggered; // Подписка
publisher.MyEvent -= subscriber.OnEventTriggered; // Отписка
```

## 4. Вызов события

Событие может быть вызвано только из класса, в котором оно объявлено.

```csharp
public class Publisher
{
    public event Action MyEvent;

    public void RaiseEvent()
    {
        MyEvent?.Invoke(); // Проверка на null перед вызовом
    }
}
```

## 5. Использование EventHandler

Встроенный делегат `EventHandler` позволяет передавать параметры в событие.

```csharp
public class EventArgsExample
{
    public event EventHandler MyEvent;

    public void RaiseEvent()
    {
        MyEvent?.Invoke(this, EventArgs.Empty);
    }
}
```

```csharp
EventArgsExample obj = new EventArgsExample();
obj.MyEvent += (sender, args) => Console.WriteLine("Событие с EventHandler!");
obj.RaiseEvent();
```

## 6. Использование EventHandler

Позволяет передавать дополнительные данные в событие с помощью класса `EventArgs`.

```csharp
public class CustomEventArgs : EventArgs
{
    public string Message { get; }
    public CustomEventArgs(string message) => Message = message;
}

public class EventExample
{
    public event EventHandler<CustomEventArgs> MyEvent;

    public void RaiseEvent(string message)
    {
        MyEvent?.Invoke(this, new CustomEventArgs(message));
    }
}
```

```csharp
EventExample example = new EventExample();
example.MyEvent += (sender, e) => Console.WriteLine($"Сообщение: {e.Message}");
example.RaiseEvent("Привет, мир!");
```

## 7. Итог

- События объявляются с ключевым словом `event`.
- Подписка осуществляется через `+=`, отписка — `-=`.
- Вызывать событие можно только из класса, где оно объявлено.
- `EventHandler<T>` используется для передачи данных вместе с событием.
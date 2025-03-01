### 1. Что такое `IOptionsMonitor` и `IOptionsSnapshot`, зачем они нужны?

В .NET Core для управления конфигурациями, помимо обычного [[IOptions]], часто используются интерфейсы `IOptionsMonitor` и `IOptionsSnapshot`. Оба интерфейса предоставляют доступ к конфигурации приложения, но с разными особенностями и назначением.

#### **`IOptionsSnapshot<T>`**

`IOptionsSnapshot<T>` предоставляет доступ к конфигурации для временных или скоупированных сервисов. Это означает, что вы можете получать конфигурацию, специфичную для текущего скоупа (например, для каждого запроса в ASP.NET Core), и конфигурация будет обновляться в течение жизни скоупа.

**Особенности:**

- Используется для получения конфигурации в рамках скоупа, например, в пределах запроса HTTP.
- Подходит для сервисов, которым нужна конфигурация, отличная от других.
- Конфигурация загружается один раз при первом доступе к `IOptionsSnapshot<T>` в скоупе.

#### **`IOptionsMonitor<T>`**

`IOptionsMonitor<T>` позволяет получать актуальное состояние конфигурации и автоматически отслеживать изменения конфигурации в реальном времени. Это полезно для приложений, которым необходимо реагировать на изменения конфигурации без перезапуска.

**Особенности:**

- Предназначен для получения конфигурации, которая может изменяться во время работы приложения.
- Используется для отслеживания изменений конфигурации и их применения без необходимости перезапуска приложения.
- Поддерживает уведомления о изменениях конфигурации.

### 2. Примеры использования с встроенными в C#/ASP.NET Core классами

#### **Пример с `IOptionsSnapshot<T>`**

Предположим, что у вас есть класс конфигурации `AppSettings`:

``` csharp
public class AppSettings
{
    public string AppName { get; set; }
    public int MaxItems { get; set; }
}
```

В `Program.cs` вы регистрируете конфигурацию:

``` csharp
builder.Services.Configure<AppSettings>(Configuration.GetSection("AppSettings"));
```

Затем в сервисе, например, в контроллере, вы используете `IOptionsSnapshot<T>` для получения конфигурации:

``` csharp
public class MyService
{
    private readonly IOptionsSnapshot<AppSettings> _settings;

    public MyService(IOptionsSnapshot<AppSettings> settings)
    {
        _settings = settings;
    }

    public void PrintConfig()
    {
        var config = _settings.Value;
        Console.WriteLine($"AppName: {config.AppName}, MaxItems: {config.MaxItems}");
    }
}
```

`IOptionsSnapshot` полезен, когда конфигурация зависит от скоупа. Если конфигурация изменяется между запросами (например, в процессе изменения настроек через API), то она будет актуальной для каждого запроса.

#### **Пример с `IOptionsMonitor<T>`**

Если вам нужно отслеживать изменения конфигурации в реальном времени, используйте `IOptionsMonitor<T>`. Например, если вы хотите отслеживать изменения в файле конфигурации, который может изменяться в процессе работы приложения:

``` csharp
public class MyService
{
    private readonly IOptionsMonitor<AppSettings> _optionsMonitor;

    public MyService(IOptionsMonitor<AppSettings> optionsMonitor)
    {
        _optionsMonitor = optionsMonitor;
        _optionsMonitor.OnChange(OnOptionsChanged);
    }

    private void OnOptionsChanged(AppSettings newSettings)
    {
        Console.WriteLine($"Configuration changed. New AppName: {newSettings.AppName}, MaxItems: {newSettings.MaxItems}");
    }

    public void PrintConfig()
    {
        var config = _optionsMonitor.CurrentValue;
        Console.WriteLine($"Current AppName: {config.AppName}, MaxItems: {config.MaxItems}");
    }
}
```

Здесь `IOptionsMonitor<T>` используется для отслеживания изменений в конфигурации. Каждый раз, когда конфигурация меняется (например, в результате изменений в `appsettings.json` или других конфигурационных источниках), метод `OnChange` будет вызываться.

### 3. Когда и как реализовывать самостоятельно

#### **Использование `IOptionsSnapshot<T>`**

- Используйте `IOptionsSnapshot<T>`, когда вам нужно получить конфигурацию в пределах скоупа (например, в течение одного HTTP-запроса).
- Это подходит для временных данных конфигурации, таких как настройки, которые зависят от сеанса или конкретного запроса пользователя.

**Реализация собственного сервиса с `IOptionsSnapshot<T>`:**

1. Создайте класс конфигурации:

``` csharp
public class AppSettings
{
    public string AppName { get; set; }
    public string Version { get; set; }
}
```

2. Зарегистрируйте конфигурацию в `ConfigureServices`:

``` csharp
builder.Services.Configure<AppSettings>(Configuration.GetSection("AppSettings"));
```

3. Внедрите `IOptionsSnapshot` в ваш сервис:

``` csharp
public class MyService
{
    private readonly IOptionsSnapshot<AppSettings> _settings;

    public MyService(IOptionsSnapshot<AppSettings> settings)
    {
        _settings = settings;
    }

    public void PrintAppInfo()
    {
        var settings = _settings.Value;
        Console.WriteLine($"AppName: {settings.AppName}, Version: {settings.Version}");
    }
}
```

#### **Использование `IOptionsMonitor<T>`**

- Используйте `IOptionsMonitor<T>`, когда вам нужно отслеживать изменения в конфигурации во время работы приложения. Это полезно, если конфигурация может изменяться в процессе работы, и вам нужно реагировать на эти изменения.

**Реализация собственного сервиса с `IOptionsMonitor<T>`:**

1. Создайте класс конфигурации:

``` csharp
public class AppSettings
{
    public string AppName { get; set; }
    public int MaxItems { get; set; }
}
```

2. Зарегистрируйте конфигурацию в `ConfigureServices`:

``` csharp
builder.Services.Configure<AppSettings>(Configuration.GetSection("AppSettings"));
```

3. Внедрите `IOptionsMonitor` в ваш сервис:

``` csharp
public class MyService
{
    private readonly IOptionsMonitor<AppSettings> _optionsMonitor;

    public MyService(IOptionsMonitor<AppSettings> optionsMonitor)
    {
        _optionsMonitor = optionsMonitor;
        _optionsMonitor.OnChange(OnOptionsChanged);
    }

    private void OnOptionsChanged(AppSettings newSettings)
    {
        Console.WriteLine($"Configuration changed: AppName: {newSettings.AppName}, MaxItems: {newSettings.MaxItems}");
    }

    public void PrintAppInfo()
    {
        var settings = _optionsMonitor.CurrentValue;
        Console.WriteLine($"AppName: {settings.AppName}, MaxItems: {settings.MaxItems}");
    }
}
```

В этом примере `IOptionsMonitor` позволяет отслеживать изменения конфигурации в реальном времени и реагировать на них с помощью метода `OnChange`.

### Когда использовать

- **`IOptionsSnapshot<T>`**: когда необходимо получать конфигурацию, специфичную для скоупа (например, для каждого HTTP-запроса), и эта конфигурация не изменяется в реальном времени.
- **`IOptionsMonitor<T>`**: когда требуется отслеживание изменений конфигурации в реальном времени и реакция на эти изменения.

Оба интерфейса облегчают работу с конфигурациями в ASP.NET Core, позволяя вам гибко управлять настройками и реагировать на их изменения.
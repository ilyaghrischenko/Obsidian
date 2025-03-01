## 1. Что это и зачем?

**Serilog** — это библиотека для логирования, которая предоставляет гибкие и расширяемые возможности для записи логов. Она поддерживает структурированное логирование, что означает, что данные о событиях логируются в виде ключ-значение, что позволяет легко анализировать и фильтровать логи. Serilog интегрируется с различными хранилищами, такими как файлы, базы данных, консоль, внешние системы и т. д.

Зачем использовать Serilog?
- Структурированное логирование, которое делает логи удобными для анализа.
- Поддержка множества хранилищ логов (файлы, базы данных, облачные сервисы и т.д.).
- Простота настройки и интеграции с ASP.NET Core.
- Возможность динамической настройки логирования в зависимости от уровня важности.

## 2. NuGet пакеты

Для того чтобы использовать Serilog в проекте, необходимо установить несколько NuGet пакетов. Наиболее важные из них:

1. **Serilog** - основной пакет для логирования.
    ```bash
    dotnet add package Serilog
    ```

2. **Serilog.Extensions.Logging** - пакет для интеграции с системами логирования ASP.NET Core.
    ```bash
    dotnet add package Serilog.Extensions.Logging
    ```

3. **Serilog.Sinks.Console** - для записи логов в консоль (можно добавить другие сinks для записи логов в другие хранилища).
    ```bash
    dotnet add package Serilog.Sinks.Console
    ```

4. **Serilog.Sinks.File** - для записи логов в файлы.
    ```bash
    dotnet add package Serilog.Sinks.File
    ```

Также, в зависимости от вашего проекта, могут понадобиться дополнительные пакеты для других источников или фильтров.

## 3. Подключение Serilog в ASP.NET Core

После установки нужных пакетов, нужно настроить Serilog в вашем приложении.

1. В **Program.cs** подключите Serilog:

```csharp
Log.Logger = new LoggerConfiguration()
            .WriteTo.Console()
            .WriteTo.File("../logs/log.txt",
                rollingInterval: RollingInterval.Day,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Error)
            .CreateLogger();

        builder.Host.UseSerilog();
        builder.Services.AddLogging();
```

2. В этом примере мы настраиваем Serilog для записи логов в консоль и файл. Логи будут сохраняться в файле, и каждый день будет создаваться новый файл.

## 4. Настройки Serilog

Serilog позволяет конфигурировать множество параметров. Некоторые из основных настроек:

### 1. **Выбор уровня логирования**

Serilog поддерживает несколько уровней логирования: `Verbose`, `Debug`, `Information`, `Warning`, `Error`, `Fatal`. Вы можете настроить минимальный уровень логирования, который будет записываться:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information() // Устанавливаем минимальный уровень для записи логов
    .WriteTo.Console()
    .WriteTo.File("logs/log.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();
```

### 2. Форматирование сообщений

Serilog позволяет настроить форматирование логов. Вы можете использовать структуру JSON или настроить вывод в другом формате:

``` csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console(new Serilog.Formatting.Compact.RenderedCompactJsonFormatter())
    .CreateLogger();
```

### 3. Добавление дополнительных данных

Вы можете добавить дополнительные данные в каждый лог (например, информацию о пользователе или о запросах), используя свойства:

``` csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.WithProperty("Application", "MyApp")
    .WriteTo.Console()
    .CreateLogger();
```

### 4. Конфигурация через файл appsettings.json

Serilog также поддерживает конфигурацию через файл `appsettings.json`. Для этого нужно добавить соответствующий пакет:

``` bash
dotnet add package Serilog.Settings.Configuration
```

И добавить настройки в `appsettings.json`:

``` json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
    "MinimumLevel": "Information",
    "WriteTo": [
      { "Name": "Console" },
      { "Name": "File", "Args": { "path": "logs/log.txt", "rollingInterval": "Day" } }
    ]
  }
}
```

Затем в `Program.cs` загрузите настройки из файла:

``` csharp
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(configuration) // Чтение конфигурации из appsettings.json
    .CreateLogger();
```

Теперь вы можете управлять настройками Serilog без необходимости изменять код, просто редактируя файл конфигурации.

## Заключение

Serilog — это мощный инструмент для логирования в ASP.NET Core, который предоставляет гибкость в настройке и ведении логов. Он поддерживает структурированные логи, различные хранилища и позволяет легко интегрировать логирование в ваше приложение.
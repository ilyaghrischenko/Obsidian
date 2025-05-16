## Что такое Hangfire?

**Hangfire** — это библиотека для .NET, предназначенная для фоновой обработки задач (background jobs). Она позволяет запускать задачи асинхронно, в отложенном режиме или по расписанию без необходимости писать дополнительный инфраструктурный код.

## Основные возможности:

- **Поддержка фоновых задач** (fire-and-forget)
    
- **Отложенные задачи** (delayed jobs)
    
- **Повторяющиеся задачи** (recurring jobs)
    
- **Задачи с продолжением** (continuations)
    
- **Обработка очередей с несколькими воркерами**
    
- **Устойчивость** — задачи сохраняются в базе данных и перезапускаются при сбоях
    
- **Интеграция с ASP.NET Core и .NET Worker Service**
    

## Где используется:

Hangfire идеально подходит для:

- Отправки email-уведомлений
    
- Генерации отчетов
    
- Очистки данных
    
- Синхронизации данных между сервисами
    
- Выполнения задач по расписанию (например, каждый день в 02:00)
    

## Архитектура и компоненты:

- **Client** — постановка задач в очередь
    
- **Storage** — место хранения задач (поддерживает SQL Server, PostgreSQL, Redis и др.)
    
- **Server** — фоновые воркеры, обрабатывающие задачи
    
- **Dashboard** — UI для мониторинга задач (доступен по /hangfire)
    

## Пример конфигурации с PostgreSQL

```csharp
string connectionString = EnvironmentExtensions.GetEnvironmentVariableOrThrowException("DB_CLOUD");

builder.Services.AddHangfire(config => config
    .UsePostgreSqlStorage(options => {
        options.UseNpgsqlConnection(connectionString);
    }));

builder.Services.AddHangfireServer();
```

## Как использовать

```csharp
// Fire-and-forget задача
BackgroundJob.Enqueue(() => Console.WriteLine("Привет, Hangfire!"));

// Задача с задержкой
BackgroundJob.Schedule(() => Console.WriteLine("Через 1 минуту"), TimeSpan.FromMinutes(1));

// Повторяющаяся задача
RecurringJob.AddOrUpdate("ежедневная-задача", () => SomeService.RunTask(), Cron.Daily);

// Задача с продолжением
var jobId = BackgroundJob.Enqueue(() => Step1());
BackgroundJob.ContinueJobWith(jobId, () => Step2());
```

## Мониторинг через Dashboard

Добавить в `Program.cs` или `Startup.cs`:

```csharp
app.UseHangfireDashboard("/hangfire");
```

## Цены

Hangfire — **open-source**, но также доступна **платная версия Hangfire Pro** с дополнительным функционалом:

- Расширенные очереди
    
- Поддержка батчей
    
- Приоритеты задач
    
- Расширенные метрики и мониторинг
    

**Pro** лицензия от $399/год.

## Заключение

Hangfire — мощное решение для управления фоновыми задачами в .NET-приложениях. Он прост в использовании, устойчив к сбоям и хорошо интегрируется с экосистемой .NET. Благодаря расширяемости и встроенному Dashboard, это одно из лучших решений для фоновой обработки в ASP.NET Core.
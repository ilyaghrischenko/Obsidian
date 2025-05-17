# Работа с .env файлами в ASP.NET Core Web API с использованием DotNetEnv

## Что такое `.env` файл?

`.env` файл — это простой текстовый файл, содержащий переменные окружения в формате:

```
KEY=VALUE
```

Эти переменные удобно использовать для хранения конфиденциальной информации (например, ключей доступа, строк подключения), особенно в **локальной разработке**.

## Подключение библиотеки DotNetEnv

Для работы с `.env` файлами можно использовать библиотеку [DotNetEnv](https://www.nuget.org/packages/DotNetEnv/):

### Установка:

```bash
dotnet add package DotNetEnv
```

## Использование в ASP.NET Core Web API

### 1. Создайте `.env` файл в корне проекта

Пример содержимого:

```
BLOB_STORAGE=DefaultEndpointsProtocol=https;AccountName=...
API_KEY=secret-key-here
```

### 2. Загрузите `.env` файл в `Program.cs`

```csharp
using DotNetEnv;

Env.Load(); // Загрузка переменных из .env
```

### 3. Получайте значения через `Environment`

```csharp
string? connectionString = Environment.GetEnvironmentVariable("BLOB_STORAGE");
```

### Важно:

- **DotNetEnv** просто копирует значения из `.env` в `Environment`.
    
- Лучше загружать `.env` **в начале запуска приложения**, до использования этих переменных.
    

## Удобный extension для работы с Environment variables

``` csharp
public static class EnvironmentExtensions
{
    public static string GetEnvironmentVariableOrThrowException(string key)
    {
        var value = Environment.GetEnvironmentVariable(key);

        if (value is null)
        {
            throw new EnvVariableNotFoundException("Environment variable not found: " + key, key);
        }

        return value;
    }
}
```

## Пример использования в репозитории

```csharp
public class BlobRepository : IBlobRepository
{
    private readonly BlobServiceClient _blobServiceClient;

    public BlobRepository()
    {
        string? connectionString = Environment.GetEnvironmentVariable("BLOB_STORAGE");
        _blobServiceClient = new BlobServiceClient(connectionString);
    }
    // ...
}
```

## Безопасность и работа в продакшене

- `.env` файл **не должен попадать в репозиторий**. Добавьте его в [[gitignore ASP NET Core]].
    
- Использование `.env` — **только для локальной разработки**.
    
- В продакшене переменные окружения задаются:
    
    - Через **конфигурацию окружения в Azure App Service / [[Azure Functions]]**
        
    - Или в Docker/Kubernetes конфигурации
        

Это более безопасный подход: переменные недоступны в коде и скрыты от разработчиков, не имеющих доступа к продакшн-среде.

## Заключение

`.env` с DotNetEnv — удобный способ управления конфиденциальными переменными при локальной разработке. Для продакшена предпочтительно использовать переменные окружения на уровне хостинга (например, Azure), чтобы обеспечить безопасность и масштабируемость.
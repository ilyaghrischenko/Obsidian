### 1. Что такое `IDisposable` и `IAsyncDisposable`, зачем они нужны?

**`IDisposable`** — это интерфейс в .NET, который предоставляет метод `Dispose()`. Он используется для освобождения ресурсов, таких как неуправляемая память или дескрипторы файлов, которые нельзя автоматически очистить сборщиком мусора.

Когда объект, использующий неуправляемые ресурсы (например, файлы, сетевые соединения или базы данных), больше не нужен, вызов `Dispose()` позволяет очистить эти ресурсы, прежде чем объект будет уничтожен сборщиком мусора. Это необходимо для предотвращения утечек памяти или других проблем с ресурсами.

**`IAsyncDisposable`** — это асинхронный аналог `IDisposable`. Он используется для асинхронной очистки ресурсов. Это важно, когда освобождение ресурса требует выполнения асинхронных операций (например, закрытие сетевых соединений или базы данных).

Метод `Dispose()` синхронен, а `DisposeAsync()` асинхронен и позволяет работать с асинхронными ресурсами без блокировки потока.

### 2. Примеры со встроенными в C#/ASP.NET Core классами

#### Пример с `IDisposable`

Пример использования `IDisposable` с классом `FileStream`:

``` csharp
public void ReadFile(string path)
{
    using (var stream = new FileStream(path, FileMode.Open))
    {
        var reader = new StreamReader(stream);
        string content = reader.ReadToEnd();
        Console.WriteLine(content);
    }
}
```

Здесь `using` гарантирует, что метод `Dispose()` будет вызван автоматически, когда объект `FileStream` выходит из области видимости.

#### Пример с `IAsyncDisposable`

Пример использования `IAsyncDisposable` с классом `HttpClient` в асинхронной среде:

``` csharp
public async Task FetchDataAsync(string url)
{
    await using (var client = new HttpClient())
    {
        var response = await client.GetStringAsync(url);
        Console.WriteLine(response);
    }
}
```

Метод `DisposeAsync()` будет вызван асинхронно, обеспечивая корректное завершение работы с сетевыми ресурсами.

### 3. Примеры, как реализовать самостоятельно и когда это требуется

#### Реализация `IDisposable`

Когда вы создаете свой класс, который использует неуправляемые ресурсы, вам нужно реализовать интерфейс `IDisposable`. Например, если ваш класс открывает файл или соединение с базой данных, вы должны обеспечить правильное освобождение этих ресурсов.

``` csharp
public class MyDatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed = false;

    public MyDatabaseConnection(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }

    public void Dispose()
    {
        if (!_disposed)
        {
            _connection?.Dispose();  // Освобождаем ресурсы
            _disposed = true;
        }
    }
}
```

Здесь класс `MyDatabaseConnection` использует `IDisposable` для очистки ресурсов соединения с базой данных.

#### Реализация `IAsyncDisposable`

Если ваш класс выполняет асинхронные операции, например, работу с файловой системой или сетью, то лучше использовать `IAsyncDisposable`:

``` csharp
public class MyAsyncService : IAsyncDisposable
{
    private readonly HttpClient _client;
    private bool _disposed = false;

    public MyAsyncService()
    {
        _client = new HttpClient();
    }

    public async Task FetchDataAsync(string url)
    {
        var response = await _client.GetStringAsync(url);
        Console.WriteLine(response);
    }

    public async ValueTask DisposeAsync()
    {
        if (!_disposed)
        {
            await _client?.DisposeAsync();  // Асинхронно освобождаем ресурсы
            _disposed = true;
        }
    }
}
```

### Когда требуются `IDisposable` и `IAsyncDisposable`

- **`IDisposable`** — если ваш класс использует ресурсы, которые необходимо явно освободить, например, файловые потоки, сетевые соединения, базы данных.
- **`IAsyncDisposable`** — если ваш класс выполняет асинхронные операции при освобождении ресурсов, такие как асинхронные запросы к серверу или закрытие асинхронных потоков.

#### Примечания:

- Использование интерфейса `IDisposable` рекомендуется при работе с ресурсами, которые не управляются сборщиком мусора.
- Интерфейс `IAsyncDisposable` следует использовать, если в процессе освобождения ресурсов необходимо выполнить асинхронные операции, чтобы не блокировать основной поток.
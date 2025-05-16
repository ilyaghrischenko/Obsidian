## Общие сведения

Azure Blob Storage — это один из типов хранилищ в Azure, предназначенный для хранения больших объемов неструктурированных данных, таких как изображения, видео, архивы, документы и др.

## Типы Blob'ов

- **Block Blob** — для обычных файлов (default).
    
- **Append Blob** — для записи данных в конец (логи).
    
- **Page Blob** — для работы с размером до 8 TB (обычно VHD).
    

---

## Важно: Работа с Azure.Storage.Blobs

```bash
Install-Package Azure.Storage.Blobs
```

## Класс для работы с BlobStorage

```csharp
public interface IBlobRepository
{
    Task<string> AddFileAndGetUrlAsync(string containerName, string fileName, Stream fileStream, CancellationToken ct);
    string GetFileUrl(string containerName, string fileName, CancellationToken ct);
    Task DeleteFileAsync(string containerName, string fileName, CancellationToken ct);
    Task DeleteAllFilesByNameAsync(string containerName, string fileName, CancellationToken ct);
}
```

```csharp
public class BlobRepository : IBlobRepository
{
    private readonly BlobServiceClient _blobServiceClient;

    public BlobRepository()
    {
        string? connectionString = Environment.GetEnvironmentVariable("BLOB_STORAGE");
        _blobServiceClient = new BlobServiceClient(connectionString);
    }

    public async Task<string> AddFileAndGetUrlAsync(string containerName, string fileName, Stream fileStream, CancellationToken ct)
    {
        BlobContainerClient container = _blobServiceClient.GetBlobContainerClient(containerName);
        await container.CreateIfNotExistsAsync(cancellationToken: ct);

        BlobClient blobClient = container.GetBlobClient(fileName);
        await blobClient.UploadAsync(fileStream, overwrite: true, cancellationToken: ct);

        await fileStream.DisposeAsync();
        return blobClient.Uri.ToString();
    }

    public string GetFileUrl(string containerName, string fileName, CancellationToken ct)
    {
        BlobContainerClient container = _blobServiceClient.GetBlobContainerClient(containerName);
        BlobClient blob = container.GetBlobClient(fileName);
        return blob.Uri.ToString();
    }

    public async Task DeleteFileAsync(string containerName, string fileName, CancellationToken ct)
    {
        BlobContainerClient container = _blobServiceClient.GetBlobContainerClient(containerName);
        BlobClient blobClient = container.GetBlobClient(fileName);
        await blobClient.DeleteIfExistsAsync(cancellationToken: ct);
    }

    public async Task DeleteAllFilesByNameAsync(string containerName, string fileName, CancellationToken ct)
    {
        BlobContainerClient container = _blobServiceClient.GetBlobContainerClient(containerName);

        await foreach (BlobItem blobItem in container.GetBlobsAsync(cancellationToken: ct))
        {
            string blobNameWithoutExtension = Path.GetFileNameWithoutExtension(blobItem.Name);

            if (blobNameWithoutExtension != fileName) continue;

            BlobClient blobClient = container.GetBlobClient(blobItem.Name);
            await blobClient.DeleteIfExistsAsync(cancellationToken: ct);
        }
    }
}
```

## Обзор основных классов

- `BlobServiceClient` — клиент для работы с blob service.
    
- `BlobContainerClient` — работа с контейнерами.
    
- `BlobClient` — для операций с конкретным blob.
    

---

## Регистрация сервиса в DI

```csharp
builder.Services.AddScoped<IBlobRepository, BlobRepository>();
```

## Пример взаимодействия в API

```csharp
[HttpPost("upload")]
public async Task<IActionResult> UploadFile(IFormFile file, [FromServices] IBlobRepository blobRepository)
{
    if (file == null || file.Length == 0)
        return BadRequest("File is empty");

    await using var stream = file.OpenReadStream();
    string url = await blobRepository.AddFileAndGetUrlAsync("mycontainer", file.FileName, stream, CancellationToken.None);

    return Ok(url);
}
```

---

## Итог

- Azure Blob Storage отлично подходит для хранения файлов и медиа.
    
- Используй BlobClient/сервис/контейнер классы для полноценной работы.
    
- Лучшая практика — создать абстракцию (BlobRepository).
tags: [#backend, #database, #mongodb, #mongodb-driver]

Краткое руководство по интеграции, настройке и best practices.

## 1. Установка

``` bash
dotnet add package MongoDB.Driver
```

## 2. Конфигурация (appsettings.json)

Разделяйте строку подключения и имя базы данных.

``` json
{
  "MongoDB": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "my_service_db"
  }
}
```

## 3. Регистрация в DI (Program.cs)

**Критически важно:** Регистрировать конвенции и сериализаторы **до** создания первого экземпляра `MongoClient` или обращения к коллекциям.

``` csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization;
using MongoDB.Bson.Serialization.Conventions;
using MongoDB.Bson.Serialization.Serializers;
using MongoDB.Driver;

// 1. Глобальные настройки сериализации
var pack = new ConventionPack
{
    new CamelCaseElementNameConvention(), // Приводит C# Property к json property (camelCase)
    new IgnoreExtraElementsConvention(true), // Игнорирует поля в БД, которых нет в классе
    new EnumRepresentationConvention(BsonType.String) // Enum храним как строки, а не int
};
ConventionRegistry.Register("GlobalConventions", pack, t => true);

// 2. Сериализаторы для специфичных типов
// Guid -> храним как стандартный UUID (иначе будет специфичный Binary subtype)
BsonSerializer.RegisterSerializer(new GuidSerializer(GuidRepresentation.Standard));
// Decimal -> храним как Decimal128 (финансовая точность), иначе будет String или Double
BsonSerializer.RegisterSerializer(new DecimalSerializer(BsonType.Decimal128));

// 3. Регистрация клиента и базы
builder.Services.AddSingleton<IMongoClient>(sp => 
    new MongoClient(builder.Configuration.GetValue<string>("MongoDB:ConnectionString")));

builder.Services.AddScoped<IMongoDatabase>(sp => 
{
    var client = sp.GetRequiredService<IMongoClient>();
    return client.GetDatabase(builder.Configuration.GetValue<string>("MongoDB:DatabaseName"));
});
```

## 4. Описание Моделей (Entities)

Используйте атрибуты для управления маппингом.

``` csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

public class Product
{
    // Маппинг на _id. 
    // Если тип string и ObjectId - драйвер сам сконвертирует.
    // Если Guid - работает благодаря GuidSerializer выше.
    [BsonId]
    public Guid Id { get; init; }

    // Обязательное поле
    public required string Name { get; set; }

    // Можно переопределить имя поля конкретно для этого свойства
    [BsonElement("cost")] 
    public decimal Price { get; set; }

    // Поля, не существующие в классе, упадут сюда
    // Удобно для паттерна Flexible Schema
    [BsonExtraElements]
    public Dictionary<string, object> Metadata { get; set; } = [];

    // Поле будет проигнорировано при записи и чтении
    [BsonIgnore]
    public string TempRuntimeValue { get; set; }
}
```

## 5. Использование (Best Practices)

Не используйте Repository Pattern поверх драйвера MongoDB, если нет специфичной логики. `IMongoCollection` — это уже репозиторий.

### Вариант А: Инжектим Database (Просто и гибко)

``` csharp
public class ProductsController(IMongoDatabase db) : ControllerBase
{
    private readonly IMongoCollection<Product> _collection = db.GetCollection<Product>("products");

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(Guid id)
    {
        // Find - основной метод поиска
        var item = await _collection.Find(x => x.Id == id).FirstOrDefaultAsync();
        return item is null ? NotFound() : Ok(item);
    }

    [HttpPost]
    public async Task<IActionResult> Create(Product product)
    {
        // InsertOneAsync не возвращает ID, он мутирует объект product
        await _collection.InsertOneAsync(product);
        return CreatedAtAction(nameof(Get), new { id = product.Id }, product);
    }
}
```

### Вариант Б: Обновление (Update Definition)

Лучше обновлять конкретные поля, а не весь документ целиком (атомарность + меньше трафика).

``` csharp
var filter = Builders<Product>.Filter.Eq(p => p.Id, id);
var update = Builders<Product>.Update
    .Set(p => p.Price, newPrice)
    .Set(p => p.UpdatedAt, DateTime.UtcNow)
    .Inc(p => p.Views, 1); // Инкремент счетчика атомарно

await _collection.UpdateOneAsync(filter, update);
```

## 6. Транзакции

Работают **только** если MongoDB запущена в режиме Replica Set (даже из одного узла). На Standalone сервере транзакции не поддерживаются.

``` csharp
using var session = await client.StartSessionAsync();
session.StartTransaction();

try 
{
    await collection1.InsertOneAsync(session, doc1);
    await collection2.UpdateOneAsync(session, filter, update);
    
    await session.CommitTransactionAsync();
}
catch 
{
    await session.AbortTransactionAsync();
}
```

## 7. Факты и подводные камни

1. **IQueryable (LINQ):** Драйвер поддерживает LINQ (`_collection.AsQueryable()`), но трансляция C# выражений в Mongo Query Language (MQL) ограничена. Если сложный LINQ падает в рантайме — переписывайте на `Builders<T>`.
    
2. **Singleton Client:** `MongoClient` потокобезопасен и должен быть Singleton. Создание нового клиента на каждый запрос убьет приложение (исчерпание сокетов).
    
3. **Null Values:** По умолчанию, если поле в классе `null`, драйвер записывает `null` в базу. Чтобы не записывать поле вообще, используйте `[BsonIgnoreIfNull]`.
    
4. **Синхронные методы:** Никогда не используйте синхронные методы (`Find`, `InsertOne`) в Web API. Это блокирует потоки. Только `*Async`.
    
5. **Индексы:** Создавайте индексы при старте приложения или в CI/CD пайплайне, а не проверяйте их наличие при каждом запросе.
    
    ``` csharp
    var indexKeys = Builders<Product>.IndexKeys.Ascending(x => x.Name);
    var indexOptions = new CreateIndexOptions { Unique = true };
    await _collection.Indexes.CreateOneAsync(new CreateIndexModel<Product>(indexKeys, indexOptions));
    ```
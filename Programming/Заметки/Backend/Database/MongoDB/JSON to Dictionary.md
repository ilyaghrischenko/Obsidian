tags: [#backend, #database, #mongodb, #json, #dictionary]

``` csharp
public static class DictionaryExtensions  
{  
    private static object? ToNative(this object? obj)  
    {        if (obj is System.Text.Json.JsonElement element)  
        {            return element.ValueKind switch  
            {  
                System.Text.Json.JsonValueKind.String => element.GetString(),  
                System.Text.Json.JsonValueKind.Number => element.TryGetInt32(out int i) ? i :   
                    element.TryGetInt64(out long l) ? l :   
                    element.GetDouble(),  
                System.Text.Json.JsonValueKind.True => true,  
                System.Text.Json.JsonValueKind.False => false,  
                System.Text.Json.JsonValueKind.Null => null,  
                _ => element.ToString()   
            };  
        }        return obj;  
    }    public static Dictionary<string, object> ToDictionaryFromJson(this Dictionary<string, object> dictionary)  
        => dictionary.ToDictionary(  
            k => k.Key,   
v => v.Value.ToNative() ?? ""  
        );  
}
```

Этот метод расширения позволяет маппить данные с json формата в обычный словарь, но для этого важно чтобы в моделе ДТО было написано так:

``` csharp
sealed record AddProductDto(  
    string Name,  
    string Description,  
    decimal Price)  
{  
    [JsonExtensionData] // !!!!!!
    public Dictionary<string, object> ExtraData { get; set; } = [];  
}
```

А модель данных должна выглядеть так:

``` csharp
public class Product  
{  
    [BsonId]  
    public Guid Id { get; set; } = Guid.NewGuid();  
  
    public string Name { get; set; } = null!;  
    public string Description { get; set; } = null!;  
    public decimal Price { get; set; }  
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;  
  
    [BsonExtraElements]  // !!!!!!!
    public Dictionary<string, object> ExtraData { get; set; } = [];  
}
```

Таким образом ендпоинт добавления будет выглядеть вот так:

``` csharp
group.MapPost("", async (  
    IMongoDatabase db,  
    [FromBody] AddProductDto request,  
    CancellationToken cancellationToken) =>  
{  
    IMongoCollection<Product>? products = db.GetCollection<Product>("products"); 
  
    // КОНВЕРТАЦИЯ СЛОВАРЯ  
    var cleanedExtraData = request.ExtraData.ToDictionaryFromJson();  
  
    var product = new Product  
    {  
        Name = request.Name,  
        Description = request.Description,  
        Price = request.Price,  
        ExtraData = cleanedExtraData // Передаем очищенный словарь  
    };  
    await products.InsertOneAsync(product, cancellationToken: cancellationToken);  
    return Results.Created($"/products/{product.Id}", product);  
});
```


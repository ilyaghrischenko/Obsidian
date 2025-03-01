## 1. Что это и зачем?

**AutoMapper** — это библиотека для объектно-объектного маппинга, которая упрощает преобразование данных между различными типами объектов, особенно в случае, когда объекты имеют похожую структуру, но не идентичны. AutoMapper автоматически копирует данные из одного объекта в другой, что особенно полезно при работе с различными слоями приложения, такими как слой данных и слой представления.

Зачем использовать AutoMapper?
- **Упрощение кода**: Автоматическое маппирование объектов снижает необходимость вручную копировать свойства между объектами.
- **Снижение ошибок**: Поскольку библиотека занимается преобразованием данных, можно избежать ошибок, связанных с неправильным присваиванием значений.
- **Чистота и поддерживаемость кода**: Уменьшается количество повторяющегося кода, что облегчает поддержку.

## 2. NuGet пакеты

Для использования AutoMapper в ASP.NET Core, необходимо установить несколько NuGet пакетов:

### **AutoMapper** - основной пакет для работы с маппингом объектов.
    ```bash
    dotnet add package AutoMapper
    ```

### **AutoMapper.Extensions.Microsoft.DependencyInjection** - пакет для интеграции с DI контейнером в ASP.NET Core.
    ```bash
    dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
    ```

## 3. Как использовать AutoMapper?

### **Создание профилей маппинга**

   Для начала необходимо создать профили, которые будут описывать, как именно нужно маппировать объекты. Профиль - это класс, который наследуется от `Profile`, и в нем определяются правила маппинга между типами.

   Пример профиля:

``` csharp
public class NewsMappingProfile : Profile  
{  
    public NewsMappingProfile()  
    {        CreateMap<NewsFullProjection, NewsResponse>()  
            .ForMember(dest => dest.Id, opt => opt.MapFrom(src => src.Id))  
            .ForMember(dest => dest.Photo, opt => opt.MapFrom(src => src.Photo))  
            .ForMember(dest => dest.Video, opt => opt.MapFrom(src => src.Video))  
            .ForMember(dest => dest.Title, opt => opt.MapFrom(src => src.Title))  
            .ForMember(dest => dest.Description, opt => opt.MapFrom(src => src.Description))  
            .ForMember(dest => dest.Priority, opt => opt.MapFrom(src => src.Priority.ToString()))  
            .ForMember(dest => dest.Date, opt => opt.MapFrom(src => src.Date.ToShortDateString()));  
    }}
```

В этом примере создается профиль, который описывает маппинг между двумя типами: `NewsDto` и `News`.

### Использование маппера

После того как профили настроены, можно использовать AutoMapper для выполнения маппинга между объектами:

``` csharp
var newsDto = new NewsDto { Title = "News", Content = "Some content" };
var news = _mapper.Map<News>(newsDto);
```

### Подключение AutoMapper в ASP.NET Core

Для подключения AutoMapper в ASP.NET Core необходимо добавить его в контейнер зависимостей в методе `ConfigureServices` (в `Program.cs` или `Startup.cs`):

``` csharp
builder.Services.AddAutoMapper(typeof(NewsMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(NewsShortMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(ScheduleMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(ScheduleShortMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(GroupPageMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(MemberShortMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(MemberMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(SocialMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(HomeNewsMappingProfile).Assembly); builder.Services.AddAutoMapper(typeof(HomeScheduleMappingProfile).Assembly);
```

В этом примере подключены различные профили маппинга, каждый из которых будет использоваться для преобразования данных между различными типами объектов. Все профили указываются через типы классов, и AutoMapper автоматически ищет все профили в сборке.

### Примечание:

- Убедитесь, что каждый профиль наследуется от `Profile` и включает все необходимые маппинги.
- Также можно указать конкретный профиль, если нужно, например, `AddAutoMapper(typeof(MyProfile))`.

Теперь AutoMapper настроен, и вы можете использовать его для маппинга объектов в вашем приложении.
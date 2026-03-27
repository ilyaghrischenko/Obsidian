## Что такое Clean + VSA?

**Clean + VSA** - это подход, в котором архитектура проекта остается чистой по границам проектов, но прикладная логика при этом организуется по use case и feature-срезам.  

То есть снаружи это все еще похоже на [[Clean Architecture]]: отдельно `Domain`, отдельно core/application-слой, отдельно infrastructure-адаптеры. Но внутри прикладного слоя код уже удобнее раскладывать не по техническим папкам `Services`, `Helpers`, `Utils`, а по сценариям и фичам, как в [[Vertical Sliced Architecture (VSA)]].

## Зачем использовать?

- Сохраняются четкие границы между доменом, application-логикой и инфраструктурой.
- Код остается удобным для роста, потому что use case можно локализовать.
- Легче подключать разные инфраструктурные реализации без изменения core-логики.
- Подход хорошо подходит для решений, где есть несколько внешних адаптеров: БД, файловое хранилище, поисковая модель и т.д.
- Удобно сочетать с [[CQRS]], [[Mediator]] и доменной моделью из [[DDD - Domain Driven Design]].

## Как это работает?

В вашем варианте архитектура выглядит примерно так:

```text
EduGraph.sln
  EduGraph.Core
  EduGraph.Domain
  EduGraph.SharedKernel
  EduGraph.Infrastructure.SQLite
  EduGraph.Infrastructure.GoogleDrive
  EduGraph.Infrastructure.SearchModel
```

### ✅ Что означает каждый проект

- `EduGraph.Domain` - доменная модель, сущности, value objects, доменные правила.
- `EduGraph.SharedKernel` - общие базовые абстракции и примитивы, которые используются в нескольких частях решения.
- `EduGraph.Core` - прикладной слой с бизнес-сценариями, orchestration и feature/use case логикой.
- `EduGraph.Infrastructure.SQLite` - реализация persistence через SQLite.
- `EduGraph.Infrastructure.GoogleDrive` - адаптер для интеграции с Google Drive.
- `EduGraph.Infrastructure.SearchModel` - отдельная инфраструктурная реализация под поисковую или индексную модель.

Здесь хорошо видно, что у вас `Clean` выражен не только папками, а именно отдельными проектами solution.

### ✅ Где здесь VSA

`VSA` в таком подходе проявляется обычно не на уровне solution-проектов, а внутри `EduGraph.Core`.

То есть `Core` не обязан быть папкой вида:

```text
Core/
  Services/
  Interfaces/
  Validators/
  Handlers/
```

Гораздо удобнее, если он организован по фичам:

```text
EduGraph.Core/
  Features/
    Courses/
      Create/
        CreateCourseCommand.cs
        CreateCourseHandler.cs
        CreateCourseValidator.cs
      GetById/
        GetCourseByIdQuery.cs
        GetCourseByIdHandler.cs
    Files/
      UploadToGoogleDrive/
        UploadFileCommand.cs
        UploadFileHandler.cs
```

Именно это и есть `VSA`-часть: сценарий локализован, а не размазан по папкам `Commands`, `Handlers`, `DTOs`, `Mappings`.

### ⚙️ Как читается такой подход

По сути:

- `Clean` отвечает за границы зависимостей между проектами.
- `VSA` отвечает за удобную организацию прикладного кода внутри `Core`.

То есть у вас не просто "чистая архитектура", и не просто "вертикальные срезы", а комбинация:

- домен и shared abstractions вынесены отдельно;
- core не знает деталей конкретной инфраструктуры;
- инфраструктура разбита на отдельные адаптеры;
- use case внутри core удобнее хранить по feature-срезам.

### ✅ Практический пример use case

Например, в `EduGraph.Core` может лежать feature:

```csharp
public sealed record UploadMaterialCommand(Guid CourseId, Stream Content, string FileName) : IRequest<string>;
```

```csharp
public sealed class UploadMaterialHandler : IRequestHandler<UploadMaterialCommand, string>
{
    private readonly IFileStorage _fileStorage;
    private readonly AppDbContext _dbContext;

    public UploadMaterialHandler(IFileStorage fileStorage, AppDbContext dbContext)
    {
        _fileStorage = fileStorage;
        _dbContext = dbContext;
    }

    public async Task<string> Handle(UploadMaterialCommand request, CancellationToken cancellationToken)
    {
        var course = await _dbContext.Courses
            .FirstOrDefaultAsync(x => x.Id == request.CourseId, cancellationToken);

        if (course is null)
            throw new InvalidOperationException("Course not found");

        var fileId = await _fileStorage.UploadAsync(request.Content, request.FileName, cancellationToken);

        course.AttachMaterial(fileId);

        await _dbContext.SaveChangesAsync(cancellationToken);

        return fileId;
    }
}
```

Здесь:

- use case находится в `Core`;
- доменная логика остается в `Domain`;
- доступ к данным идет напрямую через контекст БД, без repository-слоя;
- контракт `IFileStorage` может жить в `Core` или `SharedKernel`;
- контекст и его конфигурация живут в инфраструктуре, например через [[IEntityTypeConfiguration<T>]];
- реальная реализация файлового хранилища находится, например, в `EduGraph.Infrastructure.GoogleDrive`.

### ✅ Почему несколько infrastructure-проектов - это хороший сигнал

Отдельные проекты `EduGraph.Infrastructure.SQLite`, `EduGraph.Infrastructure.GoogleDrive`, `EduGraph.Infrastructure.SearchModel` показывают, что инфраструктура у вас уже разделена по ответственности.

Это дает несколько плюсов:

- проще менять конкретную реализацию без касания core-логики;
- интеграции не смешиваются в один огромный `Infrastructure`;
- решение лучше масштабируется по внешним адаптерам;
- проще тестировать и подключать зависимости точечно.

Такой стиль уже близок к тому, что часто встречается в больших приложениях и даже местами пересекается с идеями [[Modular Monolith]].

### ⚠️ Где в таком подходе легко ошибиться

- Превратить `Core` в свалку из общих сервисов без feature-структуры.
- Перенести слишком много общего кода в `SharedKernel`, хотя он не является по-настоящему общим.
- Начать тянуть конкретные SQLite/Google Drive-зависимости прямо в `Core` или `Domain`.
- Добавить repository-слой просто "по шаблону", хотя достаточно прямой работы с `DbContext`.
- Делать vertical slices только формально, но хранить команды, валидаторы и handlers в разных глобальных папках.
- Размыть границу между application-моделью и инфраструктурными моделями.

### ✅ Когда такой вариант особенно удобен

- Когда домен уже не помещается в один простой Web API проект.
- Когда есть несколько инфраструктурных адаптеров с разной ответственностью.
- Когда важно сохранить чистые project boundaries.
- Когда бизнес-сценарии удобнее развивать как отдельные use case внутри core-слоя.

## Заключение

В вашем варианте **Clean + VSA** - это архитектура, где `Clean` выражен через отдельные проекты solution, а `VSA` - через организацию прикладной логики внутри `Core` по фичам и use case.  

То есть основная идея не в том, чтобы заменить [[Clean Architecture]], а в том, чтобы сделать ее практичнее: оставить чистые зависимости, отдельный `Domain`, отдельный `SharedKernel`, изолированные infrastructure-адаптеры и при этом не превращать core-слой в плоскую папку из технических абстракций.

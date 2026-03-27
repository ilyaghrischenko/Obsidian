tags: [#backend, #shared-settings, #directory-packages-props]

## Что такое Directory.Packages.props?

**`Directory.Packages.props`** - это файл для централизованного управления версиями NuGet-пакетов в `.NET`-решении. Вместо того чтобы указывать `Version` в каждом `PackageReference`, версии выносятся в одно общее место.

Это особенно полезно, когда одно и то же решение использует общие пакеты вроде [[Serilog]], [[AutoMapper]] или [[Fluent Validation]] в нескольких проектах одновременно.

## Зачем использовать?

- Все версии NuGet-пакетов хранятся в одном файле.
- Проще обновлять зависимости во всем решении.
- Снижается риск, что разные проекты используют разные версии одного пакета.
- `csproj` становятся чище и короче.
- Удобно для поддержки больших решений и стабильной сборки в [[CI CD]].

## Как это работает?

В `Directory.Packages.props` описываются версии пакетов, а в `.csproj` остается только сам `PackageReference` без версии.

### ✅ Пример `Directory.Packages.props`

```xml
<Project>
  <ItemGroup>
    <PackageVersion Include="Serilog.AspNetCore" Version="8.0.1" />
    <PackageVersion Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.1" />
    <PackageVersion Include="FluentValidation.DependencyInjectionExtensions" Version="11.9.0" />
  </ItemGroup>
</Project>
```

### ✅ Пример `csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Serilog.AspNetCore" />
    <PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" />
    <PackageReference Include="FluentValidation.DependencyInjectionExtensions" />
  </ItemGroup>
</Project>
```

### ⚙️ Почему это удобно

- Обновление версии происходит в одном месте.
- Проще ревьюить изменения зависимостей.
- Удобнее сопровождать монорепы и решения с несколькими сервисами.
- Меньше шансов получить конфликт версий между API, библиотеками и тестами.

### ⚠️ Практические замечания

- `Directory.Packages.props` отвечает именно за версии пакетов, а не за остальные build-настройки.
- Общие MSBuild-свойства лучше хранить в [[Directory.Build.props]].
- Если пакет нужен только одному проекту, его все равно можно оставить локально в конкретном `.csproj`.
- Перед массовым обновлением пакетов полезно проверить совместимость библиотек и пайплайнов сборки.

### ✅ Частая комбинация

Обычно файлы используют вместе:

```text
src/
  Directory.Build.props
  Directory.Packages.props
  Api/Api.csproj
  Application/Application.csproj
  Infrastructure/Infrastructure.csproj
```

- `Directory.Build.props` хранит общие свойства сборки.
- `Directory.Packages.props` хранит версии зависимостей.

## Заключение

**`Directory.Packages.props`** делает управление NuGet-зависимостями централизованным и предсказуемым. В проектах с несколькими `.csproj` такой подход заметно упрощает обновление пакетов, снижает количество расхождений по версиям и делает конфигурацию решения чище.

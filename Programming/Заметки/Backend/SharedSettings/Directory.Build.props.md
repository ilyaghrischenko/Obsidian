## Что такое Directory.Build.props?

**`Directory.Build.props`** - это специальный файл MSBuild, который позволяет хранить общие настройки для нескольких `.csproj` в одном месте. Он автоматически подхватывается проектами внутри директории и всех вложенных папок.

На практике файл используют, чтобы не дублировать одинаковые свойства в каждом проекте решения: `Nullable`, `ImplicitUsings`, `TreatWarningsAsErrors`, `LangVersion` и другие.

## Зачем использовать?

- Убирает дублирование одинаковых настроек между проектами.
- Позволяет централизованно управлять общими правилами сборки.
- Упрощает поддержку больших решений с несколькими `class library`, API и тестами.
- Помогает держать единый стиль конфигурации для [[ASP.NET Core MVC]], библиотек и тестовых проектов.
- Удобен для CI-сценариев и предсказуемой сборки в [[CI CD]].

## Как это работает?

Файл `Directory.Build.props` применяется автоматически до загрузки содержимого `.csproj`. Поэтому его удобно использовать именно для базовых общих свойств.

### ✅ Базовый пример

```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>
</Project>
```

После этого в отдельных `.csproj` уже не нужно повторять те же самые настройки.

### ⚙️ Практический пример для solution

```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Serilog.AspNetCore" />
    <PackageReference Include="FluentValidation.DependencyInjectionExtensions" />
  </ItemGroup>
</Project>
```

Так можно централизованно подключать пакеты и базовые настройки для проектов, где используются [[Serilog]] и [[Fluent Validation]].

### ⚠️ Что важно помнить

- Файл влияет на все проекты ниже по директории.
- Если в конкретном `.csproj` указать то же свойство, локальное значение может переопределить общее.
- Для общих `PackageReference` это удобно, но версии пакетов лучше хранить отдельно в [[Directory.Packages.props]].
- Если настроек становится слишком много, стоит разделять общие build-настройки и package management.

### ✅ Типичный сценарий

```text
src/
  Directory.Build.props
  Api/Api.csproj
  Application/Application.csproj
  Infrastructure/Infrastructure.csproj
  Tests/Tests.csproj
```

В такой структуре все проекты внутри `src` автоматически получат общие настройки.

## Заключение

**`Directory.Build.props`** нужен для централизованной настройки MSBuild-свойств и уменьшения копипаста в `.csproj`. Это особенно полезно в средних и больших `.NET`-решениях, где важно поддерживать единые правила сборки, анализа и оформления проектов.

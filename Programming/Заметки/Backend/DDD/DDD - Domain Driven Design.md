
Domain-Driven Design (DDD) — это подход к проектированию программных систем, при котором основное внимание уделяется модели предметной области и логике, выражающей бизнес-правила. Цель DDD — сделать модель предметной области основной частью архитектуры приложения и кодовой базы.

## Основные концепции DDD

### 1. Предметная область (Domain)

Это сфера деятельности или знание, которые описывает система. Например, в системе интернет-магазина — это каталог товаров, заказы, пользователи, оплата и т. д.

### 2. Модель (Model)

Набор понятий, которые описывают поведение и структуру предметной области. Модель реализуется в коде: классы, методы, отношения.

### 3. Объединение экспертов и разработчиков

DDD поощряет плотное сотрудничество между разработчиками и экспертами предметной области для создания единого языка (Ubiquitous Language).

## Структура и слои в DDD

DDD делит систему на несколько слоёв ([[Clean Architecture]]):

- **Domain Layer (Слой предметной области)** — содержит бизнес-логику и доменные сущности.
    
- **Application Layer (Слой приложения)** — реализует use-cases, оркестрирует работу слоёв.
    
- **Infrastructure Layer (Слой инфраструктуры)** — реализация технических деталей (работа с БД, API, файловой системой и т. д.).
    
- **Presentation Layer (Слой представления)** — пользовательский интерфейс или API.
    

## Основные строительные блоки DDD

### 1. Entity (Сущность)

Объект с уникальной идентичностью, которая сохраняется при изменениях состояния.

```csharp
public class Order
{
    public Guid Id { get; private set; }
    public List<OrderItem> Items { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    public void AddItem(OrderItem item) { /* бизнес-логика */ }
}
```

### 2. Value Object (Объект-значение)

Не имеет идентичности, сравнивается по значению.

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }
}
```

### 3. Aggregate (Агрегат)

Группа связанных сущностей и объектов-значений, объединённых корневой сущностью (Aggregate Root). Только через корень агрегата разрешается изменять данные внутри агрегата.

```csharp
public class ShoppingCart // Aggregate Root
{
    public Guid Id { get; private set; }
    public List<CartItem> Items { get; private set; }

    public void AddItem(Product product, int quantity)
    {
        // логика добавления товара
    }
}
```

### 4. Repository (Репозиторий)

Интерфейс для доступа к агрегатам из хранилища. Прячет детали работы с БД.

```csharp
public interface IOrderRepository
{
    Order? GetById(Guid id);
    void Save(Order order);
}
```

### 5. Domain Service (Доменный сервис)

Когда логика не относится к конкретной сущности.

```csharp
public class CurrencyConverter
{
    public Money Convert(Money amount, string toCurrency) { /* ... */ }
}
```

## DDD и базы данных

DDD не требует иметь отдельные таблицы на каждый "взгляд" на сущность. Вместо этого — одна таблица на агрегат, а различные представления реализуются через отдельные модели/DTO в разных контекстах.

Пример:

- Таблица `Dogs`
    
- Ветврач видит: вес, прививки, болезни
    
- Клиент видит: кличка, порода, цвет
    
- Используются разные агрегаты или проекции (Read Models), но одна таблица.
    

## Bounded Context и контексты

Bounded Context — граница модели. В рамках одного контекста одна и та же сущность может иметь одно поведение, в другом — другое.

Пример:

- Контекст «Ветеринар»: `Dog` с мед. информацией
    
- Контекст «Магазин»: `Dog` как товар для покупки
    

Каждый контекст имеет свою модель, логику и может даже иметь своё хранилище.

## Пример архитектуры DDD в ASP.NET Core

```
src/
├── Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Repositories/
│   ├── Services/
│   └── Enums/
├── Application/
│   ├── Interfaces/
│   ├── DTOs/
│   └── UseCases/
├── Infrastructure/
│   ├── Persistence/
│   ├── Repositories/
│   └── ExternalServices/
├── WebAPI/
│   ├── Controllers/
│   └── Models/
```

## Когда использовать DDD

DDD оправдан, если:

- У вас сложная бизнес-логика
    
- Много различных правил и сценариев
    
- Разделение ответственности между командами
    

Не стоит использовать DDD:

- В простых CRUD-приложениях
    
- Когда нет предметных экспертов
    
- При высокой неопределённости проекта
    

## Заключение

DDD помогает строить гибкую, расширяемую архитектуру, где бизнес-логика — первоклассный гражданин системы. Главное — не слепо следовать правилам, а понимать принципы и применять их, когда это оправдано.
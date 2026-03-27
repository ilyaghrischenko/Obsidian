tags: [#backend, #database, #entity-framework-core, #ientitytypeconfiguration]

## **Что это такое**

IEntityTypeConfiguration — это интерфейс из [[Entity Framework Core]], предназначенный для конфигурации сущности T с использованием Fluent API. Его реализация позволяет вынести настройки сущности (свойств, ключей, связей и т.д.) из DbContext в отдельный класс, улучшая читаемость и поддержку кода.

---

## **Зачем использовать**

- **Разделение ответственности**: конфигурация сущности не захламляет DbContext.
    
- **Переиспользуемость и масштабируемость**: удобно при большом количестве сущностей.
    
- **Явная настройка**: все настройки контролируются кодом, без атрибутов.
    

---

## **Как использовать**

  

### **1. Создание класса конфигурации:**

```
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.HasKey(p => p.Id);

        builder.Property(p => p.Name)
               .IsRequired()
               .HasMaxLength(100);

        builder.HasIndex(p => p.Name);
    }
}
```

### **2. Подключение в DbContext:**

```
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
}
```

> ✅ ApplyConfigurationsFromAssembly подключает все классы, реализующие IEntityTypeConfiguration.

---

## **Важные моменты и ситуации**


### **🔹 Первичный ключ:**

```
builder.HasKey(p => p.Id);
```

### **🔹 Свойства:**

```
builder.Property(p => p.Name)
       .IsRequired()
       .HasMaxLength(100);
```

### **🔹 Индексы:**

```
builder.HasIndex(p => p.CategoryId);
```

> 📌 Рекомендуется добавлять HasIndex для всех часто используемых внешних ключей.

  

### **🔹 Внешний ключ с навигационным свойством:**

```
builder.HasOne(p => p.Category)
       .WithMany(c => c.Products)
       .HasForeignKey(p => p.CategoryId)
       .OnDelete(DeleteBehavior.Cascade);
```

### **🔹 Внешний ключ без навигационного свойства:**

```
builder.HasOne<Category>()
       .WithMany()
       .HasForeignKey(p => p.CategoryId);
```

### **🔹 Уникальный индекс:**

```
builder.HasIndex(p => p.Slug).IsUnique();
```

### **🔹 Enum:**

```
builder.Property(p => p.Status)
       .HasConversion<string>();
```

### **🔹 Значение по умолчанию:**

```
builder.Property(p => p.CreatedAt)
       .HasDefaultValueSql("CURRENT_TIMESTAMP");
```

---

## **Советы**

- Разделяй конфигурации по классам — один файл на одну сущность.
    
- Не используй DataAnnotations, если используешь Fluent API — не смешивай стили.
    
- Используй ApplyConfigurationsFromAssembly — это уменьшает ручную работу.
    
- Не забывай про HasIndex на FK/часто запрашиваемых полях.
    

---

## **Вывод**

  

IEntityTypeConfiguration — это мощный инструмент для чистой, масштабируемой и наглядной конфигурации моделей в EF Core. Используй его с умом, и твой DbContext останется компактным и поддерживаемым.
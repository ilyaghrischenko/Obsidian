## Описание

Мы используем **Fluent Builder** для создания бургера. В этой версии каждый ингредиент представлен отдельным классом. С помощью флюентного API можно поэтапно добавлять ингредиенты в бургер и собирать готовый объект с помощью удобного синтаксиса.

## Реализация

### 1. Классы ингредиентов

Ингредиенты теперь представлены отдельными классами, каждый из которых имеет свои свойства. Например, можно создать несколько видов булочек, котлет, сыров, соусов и овощей.

``` csharp
public class Bun
{
    public string Type { get; set; }
    public double Price { get; set; }

    public Bun(string type, double price)
    {
        Type = type;
        Price = price;
    }
}

public class Patty
{
    public string Type { get; set; }
    public double Price { get; set; }

    public Patty(string type, double price)
    {
        Type = type;
        Price = price;
    }
}

public class Cheese
{
    public string Type { get; set; }
    public double Price { get; set; }

    public Cheese(string type, double price)
    {
        Type = type;
        Price = price;
    }
}

public class Vegetable
{
    public string Type { get; set; }
    public double Price { get; set; }

    public Vegetable(string type, double price)
    {
        Type = type;
        Price = price;
    }
}

public class Sauce
{
    public string Type { get; set; }
    public double Price { get; set; }

    public Sauce(string type, double price)
    {
        Type = type;
        Price = price;
    }
}
```

### 2. Класс Burger

Бургер теперь будет содержать все ингредиенты и вычислять общую цену.

``` csharp
public class Burger
{
    public Bun Bun { get; private set; }
    public List<Patty> Patties { get; private set; } = new List<Patty>();
    public List<Cheese> Cheeses { get; private set; } = new List<Cheese>();
    public List<Vegetable> Vegetables { get; private set; } = new List<Vegetable>();
    public List<Sauce> Sauces { get; private set; } = new List<Sauce>();

    public double TotalPrice => CalculateTotalPrice();

    private double CalculateTotalPrice()
    {
        double total = 0;
        if (Bun != null)
            total += Bun.Price;

        total += Patties.Sum(p => p.Price);
        total += Cheeses.Sum(c => c.Price);
        total += Vegetables.Sum(v => v.Price);
        total += Sauces.Sum(s => s.Price);

        return total;
    }

    public void ShowBurgerInfo()
    {
        Console.WriteLine("Your burger contains:");
        if (Bun != null) Console.WriteLine($"Bun: {Bun.Type} - ${Bun.Price}");
        foreach (var patty in Patties) Console.WriteLine($"Patty: {patty.Type} - ${patty.Price}");
        foreach (var cheese in Cheeses) Console.WriteLine($"Cheese: {cheese.Type} - ${cheese.Price}");
        foreach (var veg in Vegetables) Console.WriteLine($"Vegetable: {veg.Type} - ${veg.Price}");
        foreach (var sauce in Sauces) Console.WriteLine($"Sauce: {sauce.Type} - ${sauce.Price}");

        Console.WriteLine($"Total Price: ${TotalPrice}");
    }
}
```

### 3. Fluent Builder

Теперь создадим флюентный билд для создания бургера. Мы будем использовать цепочку методов, чтобы добавить ингредиенты, и возвращать сам объект `Burger`.

``` csharp
public class BurgerBuilder
{
    private Burger _burger = new Burger();

    public BurgerBuilder AddBun(string type, double price)
    {
        _burger.Bun = new Bun(type, price);
        return this;
    }

    public BurgerBuilder AddPatty(string type, double price)
    {
        _burger.Patties.Add(new Patty(type, price));
        return this;
    }

    public BurgerBuilder AddCheese(string type, double price)
    {
        _burger.Cheeses.Add(new Cheese(type, price));
        return this;
    }

    public BurgerBuilder AddVegetable(string type, double price)
    {
        _burger.Vegetables.Add(new Vegetable(type, price));
        return this;
    }

    public BurgerBuilder AddSauce(string type, double price)
    {
        _burger.Sauces.Add(new Sauce(type, price));
        return this;
    }

    public Burger Build()
    {
        return _burger;
    }
}
```

### 4. Пример использования

Теперь, с помощью Fluent API, можно создавать бургеры разной сложности и конфигурации.

``` csharp
class Program
{
    static void Main()
    {
        // Создание бургеров с разными ингредиентами
        Burger classicBurger = new BurgerBuilder()
            .AddBun("Classic Bun", 1.5)
            .AddPatty("Beef Patty", 3.0)
            .AddCheese("Cheddar Cheese", 1.2)
            .AddVegetable("Lettuce", 0.5)
            .AddVegetable("Tomato", 0.7)
            .AddSauce("Ketchup", 0.3)
            .Build();

        classicBurger.ShowBurgerInfo();

        // Создание вегетарианского бургера
        Burger veggieBurger = new BurgerBuilder()
            .AddBun("Whole Wheat Bun", 1.8)
            .AddVegetable("Lettuce", 0.5)
            .AddVegetable("Tomato", 0.7)
            .AddVegetable("Cucumber", 0.6)
            .AddSauce("Mustard", 0.3)
            .Build();

        veggieBurger.ShowBurgerInfo();
    }
}
```

### 5. Преимущества

- **Чистота и удобство**: Каждый ингредиент добавляется через цепочку методов, что делает код чистым и удобным для использования.
- **Гибкость**: Легко создавать различные вариации бургера, добавляя или убирая ингредиенты на этапе построения.
- **Модульность**: Каждый ингредиент — это отдельный класс, что позволяет легко управлять компонентами бургера и добавлять новые в будущем.

### Итог

Этот пример использования **Fluent Builder** демонстрирует, как с помощью этого паттерна можно гибко собирать сложные объекты с множеством параметров.
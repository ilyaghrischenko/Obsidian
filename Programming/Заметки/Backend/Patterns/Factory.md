## Введение

Паттерн "Фабрика" (Factory Pattern) является порождающим паттерном, который используется для создания объектов без явного указания их конкретного класса. Это позволяет скрыть логику создания объектов за интерфейсом фабрики, делая систему более гибкой и расширяемой. В C# этот паттерн часто применяется для создания различных типов объектов, которые могут быть под разными условиями.

## Пример реализации паттерна "Фабрика"

В этом примере мы реализуем фабричный паттерн для создания объектов различных типов подписок: **Ежемесячной** и **Ежегодной**.

### Шаг 1: Абстракция базового класса подписки

Мы начинаем с создания абстрактного класса `BaseMembership`, который будет определять общие свойства и методы для всех типов подписок.

``` csharp
namespace Factory.Memberships
{
    public abstract class BaseMembership
    {
        public required DateOnly From { get; set; }
        public abstract required DateOnly To { get; set; }
        public required string Description { get; set; }

        public override string ToString()
        {
            string separator = new('-', 30);
            return $"{separator}\nFrom: {From}\nTo: {To}\nDescription: {Description}\n{separator}";
        }
    }
}
```

### Шаг 2: Реализация конкретных типов подписок

Теперь создадим два класса, которые будут представлять конкретные типы подписок: **Месячная подписка** и **Годовая подписка**.

#### Месячная подписка:

``` csharp
namespace Factory.Memberships
{
    public class MonthMembership : BaseMembership
    {
        private DateOnly _to;
        
        public override required DateOnly To
        {
            get => _to;
            set
            {
                int variance = value.DayNumber - From.DayNumber;

                _to = variance switch
                {
                    < 30 => throw new ArgumentException("ending data must not be less than 30 days"),
                    > 31 => throw new ArgumentException("ending data must not be more than 30 days"),
                    _ => value
                };
            }
        }
    }
}
```

Годовая подписка:

``` csharp
namespace Factory.Memberships
{
    public class YearMembership : BaseMembership
    {
        private DateOnly _to;
        
        public override required DateOnly To
        {
            get => _to;
            set
            {
                int year = 365;
                int variance = value.DayNumber - From.DayNumber;

                if (variance < year)
                {
                    throw new ArgumentException("ending data must not be less than 365 days");
                }

                if (variance > year)
                {
                    throw new AggregateException("ending data must not be greater than 365 days");
                }

                _to = value;
            }
        }
    }
}
```

### Шаг 3: Интерфейс и абстрактная фабрика

Теперь создадим интерфейс `IFactory`, который будет определять метод для создания объектов подписки, и абстрактный класс `BaseFactory`, который будет определять логику выбора конкретной фабрики для создания подписки.

``` csharp
namespace Factory.Factories
{
    public interface IFactory
    {
        BaseMembership GetMembership(DateOnly from, DateOnly to);
    }

    public abstract class BaseFactory
    {
        public static IFactory GetFactory(DateOnly from, DateOnly to)
        {
            int days = to.DayNumber - from.DayNumber;

            return days switch
            {
                30 or 31 => new MonthMembershipFactory(),
                365 => new YearMembershipFactory(),
                _ => throw new ArgumentException($"Invalid membership duration: {days} days")
            };
        }
    }
}
```

### Шаг 4: Конкретные фабрики

Теперь создадим две фабрики: `MonthMembershipFactory` и `YearMembershipFactory`, которые будут создавать соответствующие объекты подписки.

#### Месячная фабрика:

``` csharp
namespace Factory.Factories
{
    public class MonthMembershipFactory : IFactory
    {
        public BaseMembership GetMembership(DateOnly from, DateOnly to)
        {
            return new MonthMembership
            {
                From = from,
                To = to,
                Description = "Month Membership"
            };
        }
    }
}
```

Годовая фабрика:

``` csharp
namespace Factory.Factories
{
    public class YearMembershipFactory : IFactory
    {
        public BaseMembership GetMembership(DateOnly from, DateOnly to)
        {
            return new YearMembership
            {
                From = from,
                To = to,
                Description = "Year Membership"
            };
        }
    }
}
```

### Шаг 5: Использование фабрики

Наконец, давайте используем фабричный паттерн для создания подписок в основной программе.

``` csharp
using Factory.Factories;
using Factory.Memberships;

Console.Write("Enter start date: ");
var startDate = DateOnly.Parse(Console.ReadLine());

Console.Write("Enter end date: ");
var endDate = DateOnly.Parse(Console.ReadLine());

IFactory factory = BaseFactory.GetFactory(startDate, endDate);

BaseMembership membership = factory.GetMembership(startDate, endDate);

Console.WriteLine(membership);
```

### Преимущества использования фабрики

1. **Абстракция создания объектов**: Паттерн фабрика позволяет скрыть логику создания объектов и облегчить поддержку кода.
2. **Упрощение изменения типов объектов**: При необходимости добавления новых типов подписок достаточно будет добавить новую фабрику без изменения существующего кода.
3. **Гибкость**: Создание объектов через фабричный интерфейс позволяет гибко менять способ их создания в зависимости от контекста.

## Заключение

Паттерн "Фабрика" позволяет централизовать логику создания объектов и делает код более гибким и расширяемым. В приведенном примере, использование фабрики позволяет легко создавать различные типы подписок без изменения основной логики программы.
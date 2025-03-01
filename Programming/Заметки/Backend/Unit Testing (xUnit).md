## Что такое юнит-тестирование?
Юнит-тестирование — это процесс тестирования отдельных модулей (функций, методов, классов) программы в изоляции. Оно помогает:
- Выявлять ошибки на ранних этапах
- Поддерживать качество кода
- Облегчать рефакторинг
- Улучшать документацию кода

## Установка необходимых пакетов
Для написания юнит-тестов в C# используем **xUnit** и **FluentAssertions**:

```sh
 dotnet add package xUnit
 dotnet add package FluentAssertions
```

## Создание тестового проекта
Создадим новый тестовый проект:
```sh
 dotnet new xunit -n MyProject.Tests
```

## Пример юнит-теста
Допустим, у нас есть класс `Calculator`:

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
}
```

Теперь напишем тест для метода `Add` с использованием [[Fluent Assertions]]:

```csharp
using Xunit;
using FluentAssertions;

public class CalculatorTests
{
    [Fact]
    public void Add_ShouldReturnSum_WhenGivenTwoNumbers()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        var result = calculator.Add(2, 3);

        // Assert
        result.Should().Be(5);
    }
}
```

## Тестирование исключений
Допустим, метод `Divide` выбрасывает исключение при делении на ноль:

```csharp
public class Calculator
{
    public int Divide(int a, int b)
    {
        if (b == 0) throw new DivideByZeroException("Деление на ноль запрещено");
        return a / b;
    }
}
```

Тестируем выброс исключения:

```csharp
public class CalculatorTests
{
    [Fact]
    public void Divide_ShouldThrowException_WhenDividingByZero()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        Action act = () => calculator.Divide(10, 0);

        // Assert
        act.Should().Throw<DivideByZeroException>().WithMessage("Деление на ноль запрещено");
    }
}
```

## Итог

Юнит-тестирование помогает поддерживать качество кода, а FluentAssertions делает тесты читаемыми и выразительными. Использование xUnit в связке с FluentAssertions упрощает процесс тестирования в C#.

## 1. Что такое TDD?

Test Driven Development (TDD) - это методология разработки программного обеспечения, в которой написание тестов предшествует написанию кода. Основной процесс включает три шага:

1. **Red** – Написать тест, который изначально не проходит.
2. **Green** – Написать минимальный код, который делает тест успешным.
3. **Refactor** – Улучшить код, не изменяя его поведение.

## 2. Настройка [[Unit Testing (xUnit)]] и [[Fluent Assertions]]

Для работы с TDD в .NET мы используем:

- **xUnit** – фреймворк для модульного тестирования.
- **FluentAssertions** – библиотеку для удобной проверки утверждений в тестах.

### Установка

Добавьте следующие пакеты через NuGet:

```sh
dotnet add package xunit
```

```sh
dotnet add package FluentAssertions
```

## 3. Пример TDD на xUnit + FluentAssertions

Рассмотрим процесс разработки функции сложения чисел с использованием TDD.

### Шаг 1: Написание теста (Red)

Создадим тест на ожидаемое поведение функции сложения:

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

Этот тест не скомпилируется, потому что `Calculator` еще не существует.

### Шаг 2: Написание минимального кода (Green)

Теперь создадим минимальную реализацию класса `Calculator`, чтобы тест прошел:

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
}
```

Запустим тесты:

```sh
dotnet test
```

Тест должен успешно выполниться.

### Шаг 3: Рефакторинг (Refactor)

На данном этапе можно улучшить код, например, добавить логирование или обработку ошибок, если это необходимо.

## 4. Дополнительный пример с исключениями

Допустим, мы хотим, чтобы метод выбрасывал исключение при передаче отрицательных чисел.

### Тест на исключение:

```csharp
[Fact]
public void Add_ShouldThrowException_WhenGivenNegativeNumbers()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    var act = () => calculator.Add(-1, 3);
    
    // Assert
    act.Should().Throw<ArgumentException>().WithMessage("Negative numbers are not allowed");
}
```

### Реализация метода:

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        if (a < 0 || b < 0)
            throw new ArgumentException("Negative numbers are not allowed");
        return a + b;
    }
}
```

Теперь тест снова проходит.

## 5. Заключение

TDD позволяет:

- Улучшить качество кода.
- Минимизировать количество ошибок.
- Улучшить дизайн приложения.
- Сделать код более тестируемым.

Использование xUnit и FluentAssertions делает процесс тестирования удобнее и нагляднее.
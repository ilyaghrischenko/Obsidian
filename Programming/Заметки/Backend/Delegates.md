## Что такое делегаты?

Делегаты в C# представляют собой типы, которые хранят ссылки на методы. Они позволяют передавать методы в качестве параметров, а также вызывать их асинхронно.

## Объявление делегата

```csharp
public delegate void MyDelegate(string message);
```

## Использование делегатов

1. **Создание делегата**

```csharp
public delegate void PrintMessage(string message);
```

2. **Определение метода, который соответствует делегату**

```csharp
public static void ShowMessage(string message)
{
    Console.WriteLine(message);
}
```

3. **Привязка метода к делегату**

```csharp
PrintMessage print = ShowMessage;
print("Привет, мир!");
```

## Анонимные методы

```csharp
PrintMessage print = delegate (string message)
{
    Console.WriteLine(message);
};
print("Анонимный метод");
```

## Лямбда-выражения

```csharp
PrintMessage print = message => Console.WriteLine(message);
print("Лямбда-выражение");
```

## Делегаты с несколькими методами (Multicast)

```csharp
PrintMessage print = ShowMessage;
print += message => Console.WriteLine($"Дополнительно: {message}");
print("Привет!");
```

## Встроенные делегаты `C#`

1. **Action** – делегат, который не возвращает значения.

```csharp
Action<string> action = Console.WriteLine;
action("Action пример");
```

2. **Func** – делегат, который возвращает значение.

```csharp
Func<int, int, int> sum = (a, b) => a + b;
Console.WriteLine(sum(3, 5));
```

3. **Predicate** – делегат, который возвращает `bool`.

```csharp
Predicate<int> isEven = number => number % 2 == 0;
Console.WriteLine(isEven(4));
```

## Итог

Делегаты в C# позволяют работать с методами как с объектами, передавать их в другие методы, комбинировать и применять функциональный стиль программирования. Так же, на основе делагатов создаются [[Events]].
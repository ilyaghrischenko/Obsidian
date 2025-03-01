### 1. Что такое Dapper и зачем он нужен?

**Dapper** — это микро-ORM (Object-Relational Mapping) для .NET, который позволяет легко и эффективно работать с базами данных, используя SQL-запросы. Dapper предоставляет удобный способ выполнения запросов и маппинга данных из базы данных в объекты C# без излишней сложности, присущей традиционным ORM, таким как Entity Framework.

**Зачем нужен Dapper?**

- **Простота**: Dapper минимизирует количество кода, необходимого для взаимодействия с базой данных, но при этом сохраняет контроль над SQL-запросами.
- **Производительность**: Dapper значительно быстрее, чем более сложные ORM (например, Entity Framework), поскольку он не выполняет автоматическую генерацию SQL-запросов и не пытается отслеживать изменения в объектах.
- **Гибкость**: Вы можете использовать стандартные SQL-запросы и работать с базой данных так, как вам нужно.

### 2. Примеры использования Dapper с C# и ASP.NET Core

#### Пример подключения к базе данных и выполнения запроса

Предположим, у вас есть база данных с таблицей `Users`, и вы хотите выполнить запрос на получение всех пользователей.

1. Установите пакет **Dapper** через NuGet:

``` bash
dotnet add package Dapper
```

2. Подключите Dapper в вашем проекте:

``` csharp
using Dapper;
using System.Data.SqlClient;
using System.Linq;
```

3. Пример работы с базой данных:

``` csharp
public class UserService
{
    private readonly string _connectionString;

    public UserService(string connectionString)
    {
        _connectionString = connectionString;
    }

    public IEnumerable<User> GetAllUsers()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var users = connection.Query<User>("SELECT * FROM Users").ToList();
            return users;
        }
    }
}
```

В этом примере:

- Мы создаем соединение с базой данных с использованием строки подключения.
- Выполняем запрос с помощью метода `Query<T>()`, который автоматически маппит строки из базы данных в объекты типа `User`.

#### Пример с параметризированным запросом

Dapper поддерживает параметризированные запросы, что помогает избежать SQL-инъекций.

``` csharp
public User GetUserById(int userId)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        var query = "SELECT * FROM Users WHERE Id = @Id";
        var user = connection.QuerySingleOrDefault<User>(query, new { Id = userId });
        return user;
    }
}
```

В этом примере:

- Используется параметр `@Id` для безопасной передачи значения в SQL-запрос.
- Метод `QuerySingleOrDefault<T>()` возвращает один объект или `null`, если записи с таким ID нет.

#### Пример выполнения команд (Insert, Update, Delete)

Dapper также поддерживает выполнение команд, таких как вставка, обновление и удаление данных.

``` csharp
public void AddUser(User user)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        var query = "INSERT INTO Users (Name, Email) VALUES (@Name, @Email)";
        connection.Execute(query, user);
    }
}
```

Здесь:

- Метод `Execute` используется для выполнения команд, которые не возвращают результаты (например, `INSERT`, `UPDATE`, `DELETE`).
- Мы передаем объект `user`, и Dapper автоматически сопоставляет параметры запроса с полями объекта.

### 3. Когда и как реализовывать Dapper самостоятельно

#### Пример создания репозитория с Dapper

1. Создайте интерфейс репозитория:

``` csharp
public interface IUserRepository
{
    IEnumerable<User> GetAllUsers();
    User GetUserById(int userId);
    void AddUser(User user);
    void UpdateUser(User user);
    void DeleteUser(int userId);
}
```

2. Реализуйте репозиторий с Dapper:

``` csharp
public class UserRepository : IUserRepository
{
    private readonly string _connectionString;

    public UserRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public IEnumerable<User> GetAllUsers()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            return connection.Query<User>("SELECT * FROM Users").ToList();
        }
    }

    public User GetUserById(int userId)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            return connection.QuerySingleOrDefault<User>("SELECT * FROM Users WHERE Id = @Id", new { Id = userId });
        }
    }

    public void AddUser(User user)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var query = "INSERT INTO Users (Name, Email) VALUES (@Name, @Email)";
            connection.Execute(query, user);
        }
    }

    public void UpdateUser(User user)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var query = "UPDATE Users SET Name = @Name, Email = @Email WHERE Id = @Id";
            connection.Execute(query, user);
        }
    }

    public void DeleteUser(int userId)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var query = "DELETE FROM Users WHERE Id = @Id";
            connection.Execute(query, new { Id = userId });
        }
    }
}
```

В этом примере:

- Репозиторий реализует интерфейс `IUserRepository` и использует Dapper для выполнения SQL-запросов.
- Каждая операция (получение, добавление, обновление, удаление) выполняется с использованием метода `Query` или `Execute` в зависимости от типа запроса.

### Когда использовать Dapper

- **Простота и производительность**: Dapper идеально подходит для проектов, где требуется высокая производительность и простота работы с базой данных.
- **Когда нужно больше контроля над SQL**: Dapper дает вам полный контроль над SQL-запросами, что полезно, если нужно использовать сложные запросы или оптимизировать запросы для производительности.
- **Когда не нужно отслеживать изменения объектов**: В отличие от полноценных ORM (например, Entity Framework), Dapper не отслеживает изменения в объектах, что упрощает работу с базой данных, но может быть недостатком, если вам нужно это функциональность.

### Преимущества Dapper

- **Простота использования**: Вам не нужно создавать модели, которые бы соответствовали таблицам в базе данных, как в Entity Framework.
- **Высокая производительность**: Dapper быстрее, чем более тяжелые ORM, поскольку работает напрямую с SQL-запросами.
- **Гибкость**: Подходит для любых типов запросов и операций с базой данных.

Dapper — это отличный выбор для небольших и средних проектов, где важна производительность и контроль над SQL-запросами, а также для случаев, когда не требуется сложная бизнес-логика, связанная с объектами.
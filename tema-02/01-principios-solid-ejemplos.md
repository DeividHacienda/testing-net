# Los 5 Principios SOLID

Los principios SOLID son fundamentales para escribir código testeable y mantenible en .NET. Veamos cada principio con ejemplos claros de código incorrecto (bad code) y correcto (ok code).

---

## 1. Single Responsibility Principle (SRP)
**"Una clase debe tener una única razón para cambiar"**

### ❌ BAD CODE

```csharp
// Esta clase tiene MÚLTIPLES responsabilidades
public class UserService
{
    public void RegisterUser(string email, string password)
    {
        // 1. Validación
        if (string.IsNullOrEmpty(email))
            throw new ArgumentException("Email is required");
        
        if (!email.Contains("@"))
            throw new ArgumentException("Invalid email format");

        // 2. Lógica de negocio
        var user = new User 
        { 
            Email = email, 
            Password = HashPassword(password),
            CreatedDate = DateTime.Now 
        };

        // 3. Acceso a datos
        using (var connection = new SqlConnection("connectionString"))
        {
            connection.Open();
            var command = new SqlCommand(
                "INSERT INTO Users (Email, Password, CreatedDate) VALUES (@email, @password, @date)", 
                connection);
            command.Parameters.AddWithValue("@email", user.Email);
            command.Parameters.AddWithValue("@password", user.Password);
            command.Parameters.AddWithValue("@date", user.CreatedDate);
            command.ExecuteNonQuery();
        }

        // 4. Envío de email
        var smtpClient = new SmtpClient("smtp.server.com");
        smtpClient.Send("noreply@app.com", email, 
            "Welcome!", "Thanks for registering!");

        // 5. Logging
        Console.WriteLine($"User {email} registered at {DateTime.Now}");
    }

    private string HashPassword(string password) 
    {
        // Hash logic
        return password; // Simplified
    }
}
```

**Problemas:**
- Difícil de testear (necesitas base de datos, servidor SMTP, etc.)
- Si cambia la forma de enviar emails, debes modificar esta clase
- Si cambia la validación, debes modificar esta clase
- Si cambia el acceso a datos, debes modificar esta clase
- Múltiples razones para cambiar = viola SRP

### ✅ OK CODE

```csharp
// Validación separada
public class UserValidator
{
    public ValidationResult Validate(string email, string password)
    {
        var errors = new List<string>();

        if (string.IsNullOrEmpty(email))
            errors.Add("Email is required");
        
        if (!email.Contains("@"))
            errors.Add("Invalid email format");

        if (string.IsNullOrEmpty(password) || password.Length < 6)
            errors.Add("Password must be at least 6 characters");

        return new ValidationResult 
        { 
            IsValid = errors.Count == 0, 
            Errors = errors 
        };
    }
}

// Repositorio para acceso a datos
public interface IUserRepository
{
    void Add(User user);
}

public class UserRepository : IUserRepository
{
    private readonly string _connectionString;

    public UserRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void Add(User user)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var command = new SqlCommand(
                "INSERT INTO Users (Email, Password, CreatedDate) VALUES (@email, @password, @date)", 
                connection);
            command.Parameters.AddWithValue("@email", user.Email);
            command.Parameters.AddWithValue("@password", user.Password);
            command.Parameters.AddWithValue("@date", user.CreatedDate);
            command.ExecuteNonQuery();
        }
    }
}

// Servicio de email separado
public interface IEmailService
{
    void SendWelcomeEmail(string toEmail);
}

public class EmailService : IEmailService
{
    public void SendWelcomeEmail(string toEmail)
    {
        var smtpClient = new SmtpClient("smtp.server.com");
        smtpClient.Send("noreply@app.com", toEmail, 
            "Welcome!", "Thanks for registering!");
    }
}

// Logger separado
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"{DateTime.Now}: {message}");
    }
}

// Servicio de usuario con UNA sola responsabilidad: coordinar el registro
public class UserService
{
    private readonly IUserValidator _validator;
    private readonly IUserRepository _repository;
    private readonly IEmailService _emailService;
    private readonly ILogger _logger;
    private readonly IPasswordHasher _passwordHasher;

    public UserService(
        IUserValidator validator,
        IUserRepository repository,
        IEmailService emailService,
        ILogger logger,
        IPasswordHasher passwordHasher)
    {
        _validator = validator;
        _repository = repository;
        _emailService = emailService;
        _logger = logger;
        _passwordHasher = passwordHasher;
    }

    public void RegisterUser(string email, string password)
    {
        var validationResult = _validator.Validate(email, password);
        if (!validationResult.IsValid)
            throw new ValidationException(validationResult.Errors);

        var user = new User 
        { 
            Email = email, 
            Password = _passwordHasher.Hash(password),
            CreatedDate = DateTime.Now 
        };

        _repository.Add(user);
        _emailService.SendWelcomeEmail(email);
        _logger.Log($"User {email} registered successfully");
    }
}
```

**Ventajas:**
- Cada clase tiene una única responsabilidad
- Fácil de testear con mocks
- Cambios aislados (modificar email no afecta a validación)
- Código reutilizable

---

## 2. Open/Closed Principle (OCP)
**"Las entidades de software deben estar abiertas para extensión pero cerradas para modificación"**

### ❌ BAD CODE

```csharp
public class DiscountCalculator
{
    public decimal CalculateDiscount(Customer customer, decimal amount)
    {
        // Para agregar un nuevo tipo de cliente, debes MODIFICAR esta clase
        if (customer.Type == CustomerType.Regular)
        {
            return amount * 0.05m; // 5% descuento
        }
        else if (customer.Type == CustomerType.Premium)
        {
            return amount * 0.10m; // 10% descuento
        }
        else if (customer.Type == CustomerType.VIP)
        {
            return amount * 0.20m; // 20% descuento
        }
        else if (customer.Type == CustomerType.Employee)
        {
            return amount * 0.30m; // 30% descuento
        }
        
        return 0;
    }
}

public enum CustomerType
{
    Regular,
    Premium,
    VIP,
    Employee
    // Si agregas Gold aquí, debes modificar CalculateDiscount
}
```

**Problemas:**
- Cada nuevo tipo de cliente requiere modificar la clase existente
- Viola el principio de cerrado para modificación
- Alto riesgo de introducir bugs en código existente
- Dificulta los tests (debes re-testear todo cada vez)

### ✅ OK CODE

```csharp
// Abstracción para estrategias de descuento
public interface IDiscountStrategy
{
    decimal CalculateDiscount(decimal amount);
}

// Implementaciones concretas - EXTENSIÓN sin modificación
public class RegularCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount)
    {
        return amount * 0.05m; // 5%
    }
}

public class PremiumCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount)
    {
        return amount * 0.10m; // 10%
    }
}

public class VIPCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount)
    {
        return amount * 0.20m; // 20%
    }
}

public class EmployeeDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount)
    {
        return amount * 0.30m; // 30%
    }
}

// Nueva estrategia - SOLO EXTENSIÓN, sin modificar código existente
public class GoldCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount)
    {
        return amount * 0.15m; // 15%
    }
}

// Calculadora que usa estrategias
public class DiscountCalculator
{
    private readonly IDiscountStrategy _discountStrategy;

    public DiscountCalculator(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal CalculateDiscount(decimal amount)
    {
        return _discountStrategy.CalculateDiscount(amount);
    }
}

// Uso
var calculator = new DiscountCalculator(new VIPCustomerDiscount());
var discount = calculator.CalculateDiscount(1000m); // 200
```

**Ventajas:**
- Agregar nuevos descuentos NO requiere modificar código existente
- Código existente permanece estable y testeado
- Cada estrategia es independiente y testeable
- Fácil de extender

---

## 3. Liskov Substitution Principle (LSP)
**"Los objetos de una subclase deben poder sustituir a los objetos de su superclase sin alterar el comportamiento del programa"**

### ❌ BAD CODE

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }

    public int CalculateArea()
    {
        return Width * Height;
    }
}

// Square viola LSP porque cambia el comportamiento esperado
public class Square : Rectangle
{
    private int _side;

    public override int Width
    {
        get { return _side; }
        set { _side = value; Height = value; } // ¡Modifica Height!
    }

    public override int Height
    {
        get { return _side; }
        set { _side = value; Width = value; } // ¡Modifica Width!
    }
}

// Ejemplo de violación
public class AreaCalculator
{
    public void PrintArea(Rectangle rectangle)
    {
        rectangle.Width = 5;
        rectangle.Height = 4;
        
        // Esperamos área = 20
        // Pero si es Square, área = 16 (porque Height sobrescribe Width)
        Console.WriteLine($"Area: {rectangle.CalculateArea()}");
    }
}

// Test que falla
[Test]
public void TestRectangleArea()
{
    Rectangle rect = new Square(); // Sustitución
    rect.Width = 5;
    rect.Height = 4;
    
    Assert.AreEqual(20, rect.CalculateArea()); // FALLA! Devuelve 16
}
```

**Problemas:**
- Square no puede sustituir a Rectangle sin cambiar el comportamiento
- Viola las expectativas del cliente
- El principio matemático "un cuadrado es un rectángulo" NO aplica en programación orientada a objetos
- Tests fallan cuando sustituyes la clase base

### ✅ OK CODE

```csharp
// Abstracción común
public abstract class Shape
{
    public abstract int CalculateArea();
}

// Rectangle sin comportamiento conflictivo
public class Rectangle : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public Rectangle(int width, int height)
    {
        Width = width;
        Height = height;
    }

    public override int CalculateArea()
    {
        return Width * Height;
    }
}

// Square con su propio comportamiento consistente
public class Square : Shape
{
    public int Side { get; set; }

    public Square(int side)
    {
        Side = side;
    }

    public override int CalculateArea()
    {
        return Side * Side;
    }
}

// Cliente que trabaja con la abstracción correcta
public class AreaCalculator
{
    public void PrintArea(Shape shape)
    {
        Console.WriteLine($"Area: {shape.CalculateArea()}");
    }
}

// Uso correcto
var rectangle = new Rectangle(5, 4);
var square = new Square(4);

var calculator = new AreaCalculator();
calculator.PrintArea(rectangle); // Area: 20
calculator.PrintArea(square);    // Area: 16

// Ambas implementaciones respetan su contrato
```

**Otro ejemplo: Aves que vuelan**

### ❌ BAD CODE

```csharp
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Flying...");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        // Los pingüinos no vuelan!
        throw new NotSupportedException("Penguins can't fly!");
    }
}

// Viola LSP - no puedes sustituir Bird por Penguin
public void MakeBirdFly(Bird bird)
{
    bird.Fly(); // ¡Explota si es un Penguin!
}
```

### ✅ OK CODE

```csharp
public abstract class Bird
{
    public abstract void Move();
}

public interface IFlyable
{
    void Fly();
}

public class Sparrow : Bird, IFlyable
{
    public override void Move()
    {
        Fly();
    }

    public void Fly()
    {
        Console.WriteLine("Sparrow flying...");
    }
}

public class Penguin : Bird
{
    public override void Move()
    {
        Swim();
    }

    public void Swim()
    {
        Console.WriteLine("Penguin swimming...");
    }
}

// Cliente específico para aves que vuelan
public void MakeBirdFly(IFlyable flyable)
{
    flyable.Fly(); // Solo acepta aves que vuelan
}

// Cliente genérico
public void MakeBirdMove(Bird bird)
{
    bird.Move(); // Funciona con cualquier ave
}
```

**Ventajas:**
- Cada clase cumple su contrato sin sorpresas
- La sustitución funciona correctamente
- No hay excepciones inesperadas
- Tests predecibles

---

## 4. Interface Segregation Principle (ISP)
**"Los clientes no deben verse obligados a depender de interfaces que no utilizan"**

### ❌ BAD CODE

```csharp
// Interfaz "gordita" con muchos métodos
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void GetPaid();
    void TakeMeeting();
    void WriteReport();
    void ReviewCode();
    void DeployToProduction();
}

// Robot trabajador - no come ni duerme
public class RobotWorker : IWorker
{
    public void Work() { /* Trabaja */ }
    
    public void Eat() 
    { 
        // ¡Un robot no come!
        throw new NotImplementedException(); 
    }
    
    public void Sleep() 
    { 
        // ¡Un robot no duerme!
        throw new NotImplementedException(); 
    }
    
    public void GetPaid() 
    { 
        // ¡Un robot no cobra!
        throw new NotImplementedException(); 
    }

    public void TakeMeeting() { /* Puede participar */ }
    public void WriteReport() { /* Puede escribir */ }
    public void ReviewCode() { /* Puede revisar */ }
    public void DeployToProduction() { /* Puede desplegar */ }
}

// Manager - no programa
public class Manager : IWorker
{
    public void Work() { /* Trabaja */ }
    public void Eat() { /* Come */ }
    public void Sleep() { /* Duerme */ }
    public void GetPaid() { /* Cobra */ }
    public void TakeMeeting() { /* Participa */ }
    public void WriteReport() { /* Escribe */ }
    
    public void ReviewCode() 
    { 
        // Un manager no revisa código
        throw new NotImplementedException(); 
    }
    
    public void DeployToProduction() 
    { 
        // Un manager no despliega
        throw new NotImplementedException(); 
    }
}
```

**Problemas:**
- Clases obligadas a implementar métodos que no necesitan
- Métodos lanzando NotImplementedException
- Interfaz difícil de testear (muchas dependencias)
- Acoplamiento innecesario

### ✅ OK CODE

```csharp
// Interfaces segregadas - cada una con un propósito específico
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public interface IPayable
{
    void GetPaid();
}

public interface IMeetingParticipant
{
    void TakeMeeting();
}

public interface IReportWriter
{
    void WriteReport();
}

public interface ICodeReviewer
{
    void ReviewCode();
}

public interface IDeployer
{
    void DeployToProduction();
}

// Robot - solo implementa lo que necesita
public class RobotWorker : IWorkable, IMeetingParticipant, IReportWriter, 
                           ICodeReviewer, IDeployer
{
    public void Work() { Console.WriteLine("Robot working..."); }
    public void TakeMeeting() { Console.WriteLine("Robot in meeting..."); }
    public void WriteReport() { Console.WriteLine("Robot writing report..."); }
    public void ReviewCode() { Console.WriteLine("Robot reviewing code..."); }
    public void DeployToProduction() { Console.WriteLine("Robot deploying..."); }
}

// Humano - implementa lo que necesita
public class HumanWorker : IWorkable, IFeedable, ISleepable, IPayable,
                           IMeetingParticipant, IReportWriter
{
    public void Work() { Console.WriteLine("Human working..."); }
    public void Eat() { Console.WriteLine("Human eating..."); }
    public void Sleep() { Console.WriteLine("Human sleeping..."); }
    public void GetPaid() { Console.WriteLine("Human getting paid..."); }
    public void TakeMeeting() { Console.WriteLine("Human in meeting..."); }
    public void WriteReport() { Console.WriteLine("Human writing report..."); }
}

// Developer - implementa funciones técnicas
public class Developer : IWorkable, IFeedable, ISleepable, IPayable,
                        IMeetingParticipant, ICodeReviewer, IDeployer
{
    public void Work() { Console.WriteLine("Developer coding..."); }
    public void Eat() { Console.WriteLine("Developer eating..."); }
    public void Sleep() { Console.WriteLine("Developer sleeping..."); }
    public void GetPaid() { Console.WriteLine("Developer getting paid..."); }
    public void TakeMeeting() { Console.WriteLine("Developer in meeting..."); }
    public void ReviewCode() { Console.WriteLine("Developer reviewing code..."); }
    public void DeployToProduction() { Console.WriteLine("Developer deploying..."); }
}

// Manager - solo funciones de gestión
public class Manager : IWorkable, IFeedable, ISleepable, IPayable,
                      IMeetingParticipant, IReportWriter
{
    public void Work() { Console.WriteLine("Manager managing..."); }
    public void Eat() { Console.WriteLine("Manager eating..."); }
    public void Sleep() { Console.WriteLine("Manager sleeping..."); }
    public void GetPaid() { Console.WriteLine("Manager getting paid..."); }
    public void TakeMeeting() { Console.WriteLine("Manager in meeting..."); }
    public void WriteReport() { Console.WriteLine("Manager writing report..."); }
}

// Cliente que solo necesita código desplegable
public class DeploymentService
{
    private readonly IDeployer _deployer;

    public DeploymentService(IDeployer deployer)
    {
        _deployer = deployer;
    }

    public void Deploy()
    {
        _deployer.DeployToProduction();
    }
}

// Uso
var deployService = new DeploymentService(new RobotWorker());
deployService.Deploy(); // Funciona

var deployService2 = new DeploymentService(new Developer());
deployService2.Deploy(); // Funciona

// var deployService3 = new DeploymentService(new Manager()); 
// No compila - Manager no implementa IDeployer
```

**Ventajas:**
- Clases solo dependen de lo que necesitan
- Sin métodos no implementados
- Fácil de testear (interfaces pequeñas)
- Mejor composición

---

## 5. Dependency Inversion Principle (DIP)
**"Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones"**

### ❌ BAD CODE

```csharp
// Dependencia directa de implementación concreta
public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        // Envía email usando SMTP
        var smtpClient = new SmtpClient("smtp.gmail.com");
        smtpClient.Send("from@email.com", to, subject, body);
    }
}

// Clase de alto nivel depende directamente de clase de bajo nivel
public class UserService
{
    private readonly EmailService _emailService; // ¡Dependencia concreta!

    public UserService()
    {
        _emailService = new EmailService(); // ¡Acoplamiento fuerte!
    }

    public void RegisterUser(string email, string password)
    {
        // Lógica de registro
        var user = new User { Email = email, Password = password };
        
        // Guarda en base de datos
        SaveToDatabase(user);
        
        // Envía email
        _emailService.SendEmail(email, "Welcome", "Thanks for registering!");
    }

    private void SaveToDatabase(User user)
    {
        // Acceso directo a SQL Server
        using (var connection = new SqlConnection("connectionString"))
        {
            // ...
        }
    }
}
```

**Problemas:**
- UserService está acoplado a EmailService
- Imposible testear sin enviar emails reales
- No puedes cambiar a otro proveedor de email sin modificar UserService
- Difícil de mockear
- Viola también SRP

### ✅ OK CODE

```csharp
// Abstracción - NO depende de implementación concreta
public interface IEmailService
{
    void SendEmail(string to, string subject, string body);
}

public interface IUserRepository
{
    void Save(User user);
}

// Implementaciones concretas de bajo nivel
public class SmtpEmailService : IEmailService
{
    private readonly string _smtpServer;

    public SmtpEmailService(string smtpServer)
    {
        _smtpServer = smtpServer;
    }

    public void SendEmail(string to, string subject, string body)
    {
        var smtpClient = new SmtpClient(_smtpServer);
        smtpClient.Send("from@email.com", to, subject, body);
    }
}

public class SendGridEmailService : IEmailService
{
    private readonly string _apiKey;

    public SendGridEmailService(string apiKey)
    {
        _apiKey = apiKey;
    }

    public void SendEmail(string to, string subject, string body)
    {
        // Usa SendGrid API
        var client = new SendGridClient(_apiKey);
        // ...
    }
}

public class SqlUserRepository : IUserRepository
{
    private readonly string _connectionString;

    public SqlUserRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void Save(User user)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            // Guarda en SQL Server
        }
    }
}

// Clase de alto nivel depende de ABSTRACCIONES
public class UserService
{
    private readonly IEmailService _emailService;
    private readonly IUserRepository _repository;

    // Inyección de dependencias
    public UserService(IEmailService emailService, IUserRepository repository)
    {
        _emailService = emailService;
        _repository = repository;
    }

    public void RegisterUser(string email, string password)
    {
        var user = new User { Email = email, Password = password };
        
        _repository.Save(user);
        _emailService.SendEmail(email, "Welcome", "Thanks for registering!");
    }
}

// Configuración (normalmente en Startup o Program.cs)
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Puedes cambiar implementaciones fácilmente
        services.AddScoped<IEmailService, SmtpEmailService>();
        // O: services.AddScoped<IEmailService, SendGridEmailService>();
        
        services.AddScoped<IUserRepository, SqlUserRepository>();
        services.AddScoped<UserService>();
    }
}

// Testing - Fácil con mocks
[TestFixture]
public class UserServiceTests
{
    [Test]
    public void RegisterUser_ShouldSendWelcomeEmail()
    {
        // Arrange
        var mockEmailService = new Mock<IEmailService>();
        var mockRepository = new Mock<IUserRepository>();
        var service = new UserService(mockEmailService.Object, mockRepository.Object);

        // Act
        service.RegisterUser("test@email.com", "password123");

        // Assert
        mockEmailService.Verify(x => x.SendEmail(
            "test@email.com", 
            "Welcome", 
            "Thanks for registering!"), Times.Once);
        
        mockRepository.Verify(x => x.Save(It.IsAny<User>()), Times.Once);
    }
}
```

**Ventajas:**
- UserService no depende de implementaciones concretas
- Fácil de testear con mocks
- Puedes cambiar implementaciones sin modificar UserService
- Bajo acoplamiento
- Alta cohesión

---

## Resumen: Beneficios de SOLID para Testing

| Principio | Beneficio para Testing |
|-----------|------------------------|
| **SRP** | Clases pequeñas y enfocadas son más fáciles de testear |
| **OCP** | No necesitas re-testear código existente al extender |
| **LSP** | Tests funcionan correctamente con subclases |
| **ISP** | Menos dependencias = mocks más simples |
| **DIP** | Inyección de dependencias permite mocking fácil |

---

## Ejercicio Práctico

Refactoriza el siguiente código para que cumpla con todos los principios SOLID:

```csharp
public class OrderProcessor
{
    public void ProcessOrder(int orderId)
    {
        // Obtener orden de la base de datos
        using (var connection = new SqlConnection("connectionString"))
        {
            connection.Open();
            var command = new SqlCommand($"SELECT * FROM Orders WHERE Id = {orderId}", connection);
            var reader = command.ExecuteReader();
            // ... leer datos
        }

        // Validar orden
        if (orderId <= 0)
            throw new Exception("Invalid order");

        // Calcular descuento
        decimal discount = 0;
        if (orderId > 1000)
            discount = 0.10m;
        else if (orderId > 500)
            discount = 0.05m;

        // Procesar pago
        var paypal = new PayPalService();
        paypal.ProcessPayment(orderId, 100 - (100 * discount));

        // Enviar confirmación
        var smtpClient = new SmtpClient("smtp.server.com");
        smtpClient.Send("noreply@app.com", "customer@email.com", 
            "Order Confirmed", "Your order has been processed");

        // Log
        Console.WriteLine($"Order {orderId} processed");
    }
}
```

**Pistas:**
1. Separa responsabilidades (SRP)
2. Usa estrategias para descuentos (OCP)
3. Crea interfaces para dependencias (DIP)
4. Evita interfaces gordas (ISP)

---

## Conclusión

Los principios SOLID no son reglas estrictas, sino guías para escribir código más mantenible, testeable y escalable. En el contexto de .NET y testing:

- **Facilitan el uso de mocking frameworks** como Moq
- **Mejoran la cobertura de código** al tener unidades más pequeñas
- **Reducen el acoplamiento** entre componentes
- **Permiten TDD** (Test-Driven Development) de manera natural

Aplicar SOLID desde el inicio ahorra tiempo en el largo plazo y hace que tus pruebas sean más confiables y fáciles de mantener.

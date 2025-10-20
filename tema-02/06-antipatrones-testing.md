# Anti-patrones que dificultan las pruebas

## Introducción

Los anti-patrones son soluciones comunes a problemas recurrentes que inicialmente parecen correctas, pero que a largo plazo generan más problemas de los que resuelven. En el contexto del testing, estos anti-patrones dificultan la creación de pruebas efectivas, reducen la mantenibilidad del código y comprometen la calidad del software.

## 1. Singleton

El patrón Singleton garantiza que una clase tenga una única instancia global. Aunque puede parecer útil, crea dependencias ocultas y estado compartido que complica el testing.

### Problemas

- Estado global compartido entre pruebas
- Imposibilidad de aislar componentes
- Dificultad para crear mocks o fakes
- Acoplamiento fuerte entre clases

### Ejemplo problemático

```csharp
public class DatabaseConnection
{
    private static DatabaseConnection _instance;
    private static readonly object _lock = new object();
    
    private DatabaseConnection() { }
    
    public static DatabaseConnection Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new DatabaseConnection();
                    }
                }
            }
            return _instance;
        }
    }
    
    public void ExecuteQuery(string query)
    {
        // Ejecuta consulta en base de datos real
    }
}

public class UserService
{
    public User GetUser(int id)
    {
        // Dependencia oculta del Singleton
        var db = DatabaseConnection.Instance;
        // ... usa la conexión
        return new User();
    }
}
```

### Prueba difícil

```csharp
[TestMethod]
public void GetUser_ReturnsUser()
{
    var service = new UserService();
    
    // No podemos mockear DatabaseConnection.Instance
    // Las pruebas accederán a la base de datos real
    var user = service.GetUser(1);
    
    Assert.IsNotNull(user);
}
```

### Solución

```csharp
public interface IDatabaseConnection
{
    void ExecuteQuery(string query);
}

public class DatabaseConnection : IDatabaseConnection
{
    public void ExecuteQuery(string query)
    {
        // Implementación real
    }
}

public class UserService
{
    private readonly IDatabaseConnection _db;
    
    public UserService(IDatabaseConnection db)
    {
        _db = db;
    }
    
    public User GetUser(int id)
    {
        _db.ExecuteQuery($"SELECT * FROM Users WHERE Id = {id}");
        return new User();
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void GetUser_ReturnsUser()
{
    var mockDb = new Mock<IDatabaseConnection>();
    var service = new UserService(mockDb.Object);
    
    var user = service.GetUser(1);
    
    Assert.IsNotNull(user);
    mockDb.Verify(db => db.ExecuteQuery(It.IsAny<string>()), Times.Once);
}
```

## 2. Static Cling (Apego estático)

El uso excesivo de métodos y propiedades estáticas crea dependencias que no pueden ser mockeadas ni sustituidas durante las pruebas.

### Problemas

- Imposibilidad de usar inyección de dependencias
- Estado compartido entre pruebas
- Dificulta el aislamiento
- Reduce la testeabilidad

### Ejemplo problemático

```csharp
public static class Logger
{
    private static List<string> _logs = new List<string>();
    
    public static void Log(string message)
    {
        _logs.Add($"{DateTime.Now}: {message}");
        Console.WriteLine(message);
    }
    
    public static int GetLogCount()
    {
        return _logs.Count;
    }
}

public class OrderService
{
    public void CreateOrder(Order order)
    {
        Logger.Log("Creando orden");
        
        // Lógica de negocio
        
        Logger.Log("Orden creada exitosamente");
    }
}
```

### Prueba problemática

```csharp
[TestMethod]
public void CreateOrder_LogsMessages()
{
    var service = new OrderService();
    var order = new Order { Id = 1 };
    
    // El contador de logs puede estar afectado por otras pruebas
    int initialCount = Logger.GetLogCount();
    
    service.CreateOrder(order);
    
    // No podemos verificar los mensajes específicos
    // Solo el incremento en el contador
    Assert.AreEqual(initialCount + 2, Logger.GetLogCount());
}
```

### Solución

```csharp
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

public class OrderService
{
    private readonly ILogger _logger;
    
    public OrderService(ILogger logger)
    {
        _logger = logger;
    }
    
    public void CreateOrder(Order order)
    {
        _logger.Log("Creando orden");
        
        // Lógica de negocio
        
        _logger.Log("Orden creada exitosamente");
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void CreateOrder_LogsCreationMessages()
{
    var mockLogger = new Mock<ILogger>();
    var service = new OrderService(mockLogger.Object);
    var order = new Order { Id = 1 };
    
    service.CreateOrder(order);
    
    mockLogger.Verify(l => l.Log("Creando orden"), Times.Once);
    mockLogger.Verify(l => l.Log("Orden creada exitosamente"), Times.Once);
}
```

## 3. New is Glue (Construcción directa)

Crear instancias directamente dentro de métodos usando el operador `new` genera acoplamiento fuerte y dificulta el testing.

### Problemas

- Acoplamiento a implementaciones concretas
- Imposibilidad de sustituir dependencias
- Dificulta el testing de errores
- Reduce la flexibilidad

### Ejemplo problemático

```csharp
public class PaymentProcessor
{
    public bool ProcessPayment(decimal amount)
    {
        // Construcción directa de dependencias
        var gateway = new PayPalGateway();
        var validator = new CreditCardValidator();
        var logger = new FileLogger();
        
        logger.Log($"Procesando pago de {amount}");
        
        if (!validator.IsValid())
        {
            logger.Log("Tarjeta inválida");
            return false;
        }
        
        return gateway.Charge(amount);
    }
}
```

### Prueba imposible

```csharp
[TestMethod]
public void ProcessPayment_WithInvalidCard_ReturnsFalse()
{
    var processor = new PaymentProcessor();
    
    // No podemos simular una tarjeta inválida
    // Siempre usará la validación real
    var result = processor.ProcessPayment(100m);
    
    // No podemos controlar el resultado
}
```

### Solución

```csharp
public interface IPaymentGateway
{
    bool Charge(decimal amount);
}

public interface ICardValidator
{
    bool IsValid();
}

public interface ILogger
{
    void Log(string message);
}

public class PaymentProcessor
{
    private readonly IPaymentGateway _gateway;
    private readonly ICardValidator _validator;
    private readonly ILogger _logger;
    
    public PaymentProcessor(
        IPaymentGateway gateway,
        ICardValidator validator,
        ILogger logger)
    {
        _gateway = gateway;
        _validator = validator;
        _logger = logger;
    }
    
    public bool ProcessPayment(decimal amount)
    {
        _logger.Log($"Procesando pago de {amount}");
        
        if (!_validator.IsValid())
        {
            _logger.Log("Tarjeta inválida");
            return false;
        }
        
        return _gateway.Charge(amount);
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void ProcessPayment_WithInvalidCard_ReturnsFalse()
{
    var mockGateway = new Mock<IPaymentGateway>();
    var mockValidator = new Mock<ICardValidator>();
    var mockLogger = new Mock<ILogger>();
    
    mockValidator.Setup(v => v.IsValid()).Returns(false);
    
    var processor = new PaymentProcessor(
        mockGateway.Object,
        mockValidator.Object,
        mockLogger.Object);
    
    var result = processor.ProcessPayment(100m);
    
    Assert.IsFalse(result);
    mockLogger.Verify(l => l.Log("Tarjeta inválida"), Times.Once);
    mockGateway.Verify(g => g.Charge(It.IsAny<decimal>()), Times.Never);
}
```

## 4. God Object (Objeto Dios)

Una clase que concentra demasiadas responsabilidades, conoce demasiado sobre el sistema y hace demasiadas cosas.

### Problemas

- Violación del principio de responsabilidad única
- Pruebas complejas y largas
- Múltiples razones para cambiar
- Difícil mantenimiento

### Ejemplo problemático

```csharp
public class UserManager
{
    public bool ValidateUser(string username, string password)
    {
        // Validación de credenciales
        if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
            return false;
            
        // Acceso a base de datos
        var connection = new SqlConnection("...");
        // ...
        
        return true;
    }
    
    public void SendWelcomeEmail(string email)
    {
        // Configuración de SMTP
        var client = new SmtpClient("smtp.example.com");
        // Envío de email
    }
    
    public void LogUserActivity(string username, string action)
    {
        // Escritura en archivo de log
        File.AppendAllText("log.txt", $"{username}: {action}");
    }
    
    public bool UpdateUserProfile(int userId, string name, string email)
    {
        // Validación de datos
        // Acceso a base de datos
        // Envío de email de confirmación
        // Logging de cambios
        return true;
    }
    
    public List<string> GetUserPermissions(int userId)
    {
        // Consulta a base de datos
        // Procesamiento de roles
        // Caché de permisos
        return new List<string>();
    }
}
```

### Prueba difícil

```csharp
[TestMethod]
public void UpdateUserProfile_UpdatesSuccessfully()
{
    var manager = new UserManager();
    
    // Para probar un método necesitamos:
    // - Base de datos disponible
    // - Servidor SMTP configurado
    // - Sistema de archivos con permisos
    // - Múltiples mocks
    
    var result = manager.UpdateUserProfile(1, "Juan", "juan@example.com");
    
    Assert.IsTrue(result);
}
```

### Solución

```csharp
public interface IUserValidator
{
    bool ValidateCredentials(string username, string password);
}

public interface IEmailService
{
    void SendWelcomeEmail(string email);
}

public interface IActivityLogger
{
    void LogActivity(string username, string action);
}

public interface IUserRepository
{
    bool UpdateProfile(int userId, string name, string email);
    List<string> GetPermissions(int userId);
}

public class UserService
{
    private readonly IUserValidator _validator;
    private readonly IEmailService _emailService;
    private readonly IActivityLogger _logger;
    private readonly IUserRepository _repository;
    
    public UserService(
        IUserValidator validator,
        IEmailService emailService,
        IActivityLogger logger,
        IUserRepository repository)
    {
        _validator = validator;
        _emailService = emailService;
        _logger = logger;
        _repository = repository;
    }
    
    public bool UpdateProfile(int userId, string name, string email)
    {
        var result = _repository.UpdateProfile(userId, name, email);
        
        if (result)
        {
            _emailService.SendWelcomeEmail(email);
            _logger.LogActivity(name, "Profile Updated");
        }
        
        return result;
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void UpdateProfile_WhenSuccessful_SendsEmailAndLogs()
{
    var mockValidator = new Mock<IUserValidator>();
    var mockEmail = new Mock<IEmailService>();
    var mockLogger = new Mock<IActivityLogger>();
    var mockRepo = new Mock<IUserRepository>();
    
    mockRepo.Setup(r => r.UpdateProfile(1, "Juan", "juan@example.com"))
            .Returns(true);
    
    var service = new UserService(
        mockValidator.Object,
        mockEmail.Object,
        mockLogger.Object,
        mockRepo.Object);
    
    var result = service.UpdateProfile(1, "Juan", "juan@example.com");
    
    Assert.IsTrue(result);
    mockEmail.Verify(e => e.SendWelcomeEmail("juan@example.com"), Times.Once);
    mockLogger.Verify(l => l.LogActivity("Juan", "Profile Updated"), Times.Once);
}
```

## 5. Tight Coupling (Acoplamiento fuerte)

Dependencias directas entre clases que dificultan el cambio y el testing independiente.

### Problemas

- Cambios en una clase afectan a muchas otras
- Imposible probar componentes aisladamente
- Dificulta la reutilización
- Reduce la flexibilidad

### Ejemplo problemático

```csharp
public class EmailSender
{
    public void Send(string to, string subject, string body)
    {
        Console.WriteLine($"Enviando email a {to}");
    }
}

public class NotificationService
{
    private EmailSender _emailSender;
    
    public NotificationService()
    {
        _emailSender = new EmailSender();
    }
    
    public void NotifyUser(string email, string message)
    {
        _emailSender.Send(email, "Notificación", message);
    }
}

public class OrderController
{
    private NotificationService _notificationService;
    
    public OrderController()
    {
        _notificationService = new NotificationService();
    }
    
    public void CompleteOrder(int orderId)
    {
        // Lógica de orden
        _notificationService.NotifyUser("user@example.com", "Orden completada");
    }
}
```

### Solución

```csharp
public interface IEmailSender
{
    void Send(string to, string subject, string body);
}

public class EmailSender : IEmailSender
{
    public void Send(string to, string subject, string body)
    {
        Console.WriteLine($"Enviando email a {to}");
    }
}

public interface INotificationService
{
    void NotifyUser(string email, string message);
}

public class NotificationService : INotificationService
{
    private readonly IEmailSender _emailSender;
    
    public NotificationService(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }
    
    public void NotifyUser(string email, string message)
    {
        _emailSender.Send(email, "Notificación", message);
    }
}

public class OrderController
{
    private readonly INotificationService _notificationService;
    
    public OrderController(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }
    
    public void CompleteOrder(int orderId)
    {
        // Lógica de orden
        _notificationService.NotifyUser("user@example.com", "Orden completada");
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void CompleteOrder_NotifiesUser()
{
    var mockNotification = new Mock<INotificationService>();
    var controller = new OrderController(mockNotification.Object);
    
    controller.CompleteOrder(123);
    
    mockNotification.Verify(
        n => n.NotifyUser("user@example.com", "Orden completada"),
        Times.Once);
}
```

## 6. Hidden Dependencies (Dependencias ocultas)

Dependencias que no son explícitas en la firma del constructor o método, como acceso a archivos, base de datos, o servicios externos.

### Problemas

- No es obvio qué necesita la clase
- Dificulta el testing
- Sorpresas en tiempo de ejecución
- Viola el principio de responsabilidad única

### Ejemplo problemático

```csharp
public class ReportGenerator
{
    public string GenerateReport(int userId)
    {
        // Dependencia oculta a la base de datos
        var connection = new SqlConnection(
            ConfigurationManager.ConnectionStrings["Default"].ConnectionString);
        
        // Dependencia oculta al sistema de archivos
        var template = File.ReadAllText("template.html");
        
        // Dependencia oculta a DateTime
        var reportDate = DateTime.Now;
        
        // Dependencia oculta a configuración
        var companyName = ConfigurationManager.AppSettings["CompanyName"];
        
        return $"<html>Reporte generado el {reportDate}</html>";
    }
}
```

### Solución

```csharp
public interface IUserRepository
{
    User GetUser(int userId);
}

public interface ITemplateLoader
{
    string LoadTemplate(string name);
}

public interface IDateTimeProvider
{
    DateTime Now { get; }
}

public interface IConfiguration
{
    string GetSetting(string key);
}

public class ReportGenerator
{
    private readonly IUserRepository _userRepository;
    private readonly ITemplateLoader _templateLoader;
    private readonly IDateTimeProvider _dateTimeProvider;
    private readonly IConfiguration _configuration;
    
    public ReportGenerator(
        IUserRepository userRepository,
        ITemplateLoader templateLoader,
        IDateTimeProvider dateTimeProvider,
        IConfiguration configuration)
    {
        _userRepository = userRepository;
        _templateLoader = templateLoader;
        _dateTimeProvider = dateTimeProvider;
        _configuration = configuration;
    }
    
    public string GenerateReport(int userId)
    {
        var user = _userRepository.GetUser(userId);
        var template = _templateLoader.LoadTemplate("template.html");
        var reportDate = _dateTimeProvider.Now;
        var companyName = _configuration.GetSetting("CompanyName");
        
        return $"<html>Reporte de {user.Name} generado el {reportDate} por {companyName}</html>";
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void GenerateReport_IncludesUserNameAndDate()
{
    var mockRepo = new Mock<IUserRepository>();
    var mockTemplate = new Mock<ITemplateLoader>();
    var mockDateTime = new Mock<IDateTimeProvider>();
    var mockConfig = new Mock<IConfiguration>();
    
    var testUser = new User { Id = 1, Name = "Juan Pérez" };
    var testDate = new DateTime(2024, 1, 15);
    
    mockRepo.Setup(r => r.GetUser(1)).Returns(testUser);
    mockTemplate.Setup(t => t.LoadTemplate("template.html")).Returns("<html></html>");
    mockDateTime.Setup(d => d.Now).Returns(testDate);
    mockConfig.Setup(c => c.GetSetting("CompanyName")).Returns("Mi Empresa");
    
    var generator = new ReportGenerator(
        mockRepo.Object,
        mockTemplate.Object,
        mockDateTime.Object,
        mockConfig.Object);
    
    var result = generator.GenerateReport(1);
    
    Assert.IsTrue(result.Contains("Juan Pérez"));
    Assert.IsTrue(result.Contains("2024"));
}
```

## 7. Concrete Dependencies (Dependencias concretas)

Depender de clases concretas en lugar de abstracciones viola el principio de inversión de dependencias.

### Ejemplo problemático

```csharp
public class MySqlDatabase
{
    public List<Customer> GetCustomers()
    {
        // Acceso específico a MySQL
        return new List<Customer>();
    }
}

public class CustomerService
{
    private MySqlDatabase _database;
    
    public CustomerService()
    {
        _database = new MySqlDatabase();
    }
    
    public List<Customer> GetActiveCustomers()
    {
        var customers = _database.GetCustomers();
        return customers.Where(c => c.IsActive).ToList();
    }
}
```

### Solución

```csharp
public interface IDatabase
{
    List<Customer> GetCustomers();
}

public class MySqlDatabase : IDatabase
{
    public List<Customer> GetCustomers()
    {
        // Implementación específica de MySQL
        return new List<Customer>();
    }
}

public class CustomerService
{
    private readonly IDatabase _database;
    
    public CustomerService(IDatabase database)
    {
        _database = database;
    }
    
    public List<Customer> GetActiveCustomers()
    {
        var customers = _database.GetCustomers();
        return customers.Where(c => c.IsActive).ToList();
    }
}
```

## 8. Test-by-Exception

Usar excepciones para controlar el flujo del programa en lugar de valores de retorno apropiados.

### Ejemplo problemático

```csharp
public class UserService
{
    public void ValidateAge(int age)
    {
        if (age < 18)
        {
            throw new Exception("Usuario menor de edad");
        }
        
        if (age > 120)
        {
            throw new Exception("Edad no válida");
        }
    }
    
    public void RegisterUser(User user)
    {
        try
        {
            ValidateAge(user.Age);
            // Registro exitoso
        }
        catch (Exception ex)
        {
            // Manejo de error
        }
    }
}
```

### Solución

```csharp
public class ValidationResult
{
    public bool IsValid { get; set; }
    public string ErrorMessage { get; set; }
    
    public static ValidationResult Success()
    {
        return new ValidationResult { IsValid = true };
    }
    
    public static ValidationResult Failure(string message)
    {
        return new ValidationResult { IsValid = false, ErrorMessage = message };
    }
}

public class UserService
{
    public ValidationResult ValidateAge(int age)
    {
        if (age < 18)
        {
            return ValidationResult.Failure("Usuario menor de edad");
        }
        
        if (age > 120)
        {
            return ValidationResult.Failure("Edad no válida");
        }
        
        return ValidationResult.Success();
    }
    
    public bool RegisterUser(User user)
    {
        var validation = ValidateAge(user.Age);
        
        if (!validation.IsValid)
        {
            return false;
        }
        
        // Registro exitoso
        return true;
    }
}
```

### Prueba mejorada

```csharp
[TestMethod]
public void ValidateAge_WithMinorAge_ReturnsFailure()
{
    var service = new UserService();
    
    var result = service.ValidateAge(15);
    
    Assert.IsFalse(result.IsValid);
    Assert.AreEqual("Usuario menor de edad", result.ErrorMessage);
}

[TestMethod]
public void ValidateAge_WithValidAge_ReturnsSuccess()
{
    var service = new UserService();
    
    var result = service.ValidateAge(25);
    
    Assert.IsTrue(result.IsValid);
}
```

## Conclusiones

Los anti-patrones son trampas comunes que reducen la calidad del código y dificultan el testing. Reconocerlos y evitarlos es fundamental para:

- Escribir código más testeable y mantenible
- Reducir el acoplamiento entre componentes
- Facilitar el uso de mocks y pruebas aisladas
- Mejorar la calidad general del software

Las claves para evitar estos anti-patrones son:

- Aplicar los principios SOLID consistentemente
- Usar inyección de dependencias en lugar de construcción directa
- Preferir abstracciones sobre implementaciones concretas
- Hacer explícitas todas las dependencias
- Dividir responsabilidades en clases pequeñas y enfocadas
- Evitar estado global y dependencias estáticas

Un código bien diseñado no solo es más fácil de probar, sino también más fácil de entender, mantener y evolucionar.

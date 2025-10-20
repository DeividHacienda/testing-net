# Inyección de Dependencias en .NET

## Índice
1. [Introducción](#introducción)
2. [¿Qué es la Inyección de Dependencias?](#qué-es-la-inyección-de-dependencias)
3. [Beneficios para el Testing](#beneficios-para-el-testing)
4. [Constructor Injection](#constructor-injection)
5. [Property Injection](#property-injection)
6. [Method Injection](#method-injection)
7. [Comparación de Patrones](#comparación-de-patrones)
8. [Ejemplos Prácticos](#ejemplos-prácticos)
9. [Mejores Prácticas](#mejores-prácticas)
10. [Anti-patrones Comunes](#anti-patrones-comunes)

---

## Introducción

La **Inyección de Dependencias (DI)** es un patrón de diseño fundamental que implementa el principio de **Inversión de Dependencias (DIP)** de SOLID. Permite escribir código desacoplado, mantenible y, crucialmente, **testeable**.

En .NET, la DI está integrada nativamente desde .NET Core, proporcionando un contenedor IoC (Inversion of Control) robusto que facilita la gestión del ciclo de vida de las dependencias.

---

## ¿Qué es la Inyección de Dependencias?

La Inyección de Dependencias es una técnica donde un objeto recibe otros objetos de los que depende (sus dependencias) en lugar de crearlos internamente. Esto invierte el control de la creación de objetos.

### Ejemplo SIN Inyección de Dependencias

```csharp
public class OrderService
{
    private readonly EmailService _emailService;
    
    public OrderService()
    {
        // ❌ Acoplamiento fuerte - difícil de testear
        _emailService = new EmailService();
    }
    
    public void ProcessOrder(Order order)
    {
        // Lógica de procesamiento
        _emailService.SendConfirmation(order);
    }
}
```

**Problemas:**
- Imposible testear sin enviar emails reales
- No se puede cambiar la implementación fácilmente
- Viola el principio de Inversión de Dependencias

### Ejemplo CON Inyección de Dependencias

```csharp
public interface IEmailService
{
    void SendConfirmation(Order order);
}

public class OrderService
{
    private readonly IEmailService _emailService;
    
    // ✅ La dependencia se inyecta desde fuera
    public OrderService(IEmailService emailService)
    {
        _emailService = emailService ?? throw new ArgumentNullException(nameof(emailService));
    }
    
    public void ProcessOrder(Order order)
    {
        // Lógica de procesamiento
        _emailService.SendConfirmation(order);
    }
}
```

---

## Beneficios para el Testing

La DI transforma el testing al permitir:

1. **Mocking**: Sustituir dependencias reales por mocks o fakes
2. **Aislamiento**: Testear una clase sin ejecutar código de sus dependencias
3. **Velocidad**: Tests más rápidos al evitar operaciones costosas (BD, red, etc.)
4. **Control**: Simular diferentes escenarios y casos edge

### Ejemplo de Test con DI

```csharp
[Fact]
public void ProcessOrder_ShouldSendConfirmationEmail()
{
    // Arrange
    var mockEmailService = new Mock<IEmailService>();
    var orderService = new OrderService(mockEmailService.Object);
    var order = new Order { Id = 1, Total = 100 };
    
    // Act
    orderService.ProcessOrder(order);
    
    // Assert
    mockEmailService.Verify(x => x.SendConfirmation(order), Times.Once);
}
```

---

## Constructor Injection

El método **más recomendado** y común. Las dependencias se pasan a través del constructor.

### Características

- ✅ **Inmutabilidad**: Las dependencias son `readonly`
- ✅ **Obligatoriedad**: No se puede crear la instancia sin las dependencias
- ✅ **Claridad**: Las dependencias son explícitas en la firma del constructor
- ✅ **Thread-safe**: Al ser inmutables, son seguras en entornos multi-hilo

### Ejemplo Completo

```csharp
public interface ILogger
{
    void Log(string message);
}

public interface IRepository<T>
{
    Task<T> GetByIdAsync(int id);
    Task SaveAsync(T entity);
}

public class ProductService
{
    private readonly IRepository<Product> _productRepository;
    private readonly ILogger _logger;
    private readonly IPriceCalculator _priceCalculator;
    
    // Constructor con múltiples dependencias
    public ProductService(
        IRepository<Product> productRepository,
        ILogger logger,
        IPriceCalculator priceCalculator)
    {
        _productRepository = productRepository ?? 
            throw new ArgumentNullException(nameof(productRepository));
        _logger = logger ?? 
            throw new ArgumentNullException(nameof(logger));
        _priceCalculator = priceCalculator ?? 
            throw new ArgumentNullException(nameof(priceCalculator));
    }
    
    public async Task<decimal> GetProductPriceAsync(int productId)
    {
        _logger.Log($"Fetching product {productId}");
        
        var product = await _productRepository.GetByIdAsync(productId);
        if (product == null)
        {
            throw new ProductNotFoundException(productId);
        }
        
        return _priceCalculator.Calculate(product);
    }
}
```

### Registro en el Contenedor DI

```csharp
// Program.cs o Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<IRepository<Product>, ProductRepository>();
    services.AddSingleton<ILogger, ConsoleLogger>();
    services.AddTransient<IPriceCalculator, StandardPriceCalculator>();
    services.AddScoped<ProductService>();
}
```

### Test Unitario

```csharp
public class ProductServiceTests
{
    [Fact]
    public async Task GetProductPrice_ProductExists_ReturnsCalculatedPrice()
    {
        // Arrange
        var mockRepository = new Mock<IRepository<Product>>();
        var mockLogger = new Mock<ILogger>();
        var mockCalculator = new Mock<IPriceCalculator>();
        
        var product = new Product { Id = 1, Name = "Test Product", BasePrice = 100 };
        mockRepository.Setup(x => x.GetByIdAsync(1))
            .ReturnsAsync(product);
        mockCalculator.Setup(x => x.Calculate(product))
            .Returns(120m);
        
        var service = new ProductService(
            mockRepository.Object,
            mockLogger.Object,
            mockCalculator.Object);
        
        // Act
        var price = await service.GetProductPriceAsync(1);
        
        // Assert
        Assert.Equal(120m, price);
        mockLogger.Verify(x => x.Log(It.IsAny<string>()), Times.Once);
    }
}
```

---

## Property Injection

Las dependencias se inyectan a través de propiedades públicas después de crear la instancia.

### Características

- ⚠️ **Opcional**: La dependencia puede ser null si no se asigna
- ⚠️ **Mutable**: La propiedad puede cambiar después de la creación
- ✅ **Útil para dependencias opcionales**
- ❌ **Menos preferible que Constructor Injection**

### Cuándo Usar Property Injection

1. **Dependencias opcionales** que tienen un valor por defecto razonable
2. **Propiedades de configuración** que pueden tener valores predeterminados
3. **Frameworks legacy** que no soportan constructor injection
4. **Clases con muchas dependencias** donde algunas son raramente usadas

### Ejemplo

```csharp
public class ReportGenerator
{
    private readonly IDataSource _dataSource;
    
    // Dependencia obligatoria por constructor
    public ReportGenerator(IDataSource dataSource)
    {
        _dataSource = dataSource ?? throw new ArgumentNullException(nameof(dataSource));
    }
    
    // Dependencia opcional por propiedad
    public ILogger? Logger { get; set; }
    
    // Dependencia opcional con valor por defecto
    private IFormatter _formatter = new DefaultFormatter();
    public IFormatter Formatter
    {
        get => _formatter;
        set => _formatter = value ?? throw new ArgumentNullException(nameof(value));
    }
    
    public string GenerateReport()
    {
        Logger?.Log("Starting report generation");
        
        var data = _dataSource.GetData();
        var formatted = _formatter.Format(data);
        
        Logger?.Log("Report generated successfully");
        
        return formatted;
    }
}
```

### Uso con el Contenedor DI

```csharp
// Configuración manual
var reportGenerator = serviceProvider.GetRequiredService<ReportGenerator>();
reportGenerator.Logger = serviceProvider.GetService<ILogger>();
reportGenerator.Formatter = serviceProvider.GetService<IFormatter>() ?? new DefaultFormatter();

// O usar un inicializador
services.AddScoped<ReportGenerator>(sp =>
{
    var generator = new ReportGenerator(sp.GetRequiredService<IDataSource>());
    generator.Logger = sp.GetService<ILogger>();
    return generator;
});
```

### Test con Property Injection

```csharp
[Fact]
public void GenerateReport_WithCustomLogger_LogsMessages()
{
    // Arrange
    var mockDataSource = new Mock<IDataSource>();
    var mockLogger = new Mock<ILogger>();
    
    mockDataSource.Setup(x => x.GetData()).Returns("test data");
    
    var generator = new ReportGenerator(mockDataSource.Object)
    {
        Logger = mockLogger.Object // Inyección por propiedad en el test
    };
    
    // Act
    generator.GenerateReport();
    
    // Assert
    mockLogger.Verify(x => x.Log(It.IsAny<string>()), Times.AtLeast(2));
}

[Fact]
public void GenerateReport_WithoutLogger_WorksCorrectly()
{
    // Arrange
    var mockDataSource = new Mock<IDataSource>();
    mockDataSource.Setup(x => x.GetData()).Returns("test data");
    
    var generator = new ReportGenerator(mockDataSource.Object);
    // No se asigna Logger - debería funcionar igual
    
    // Act
    var result = generator.GenerateReport();
    
    // Assert
    Assert.NotNull(result);
}
```

---

## Method Injection

Las dependencias se pasan como parámetros a métodos específicos que las necesitan.

### Características

- ✅ **Flexibilidad**: Diferentes implementaciones por llamada
- ✅ **Alcance limitado**: La dependencia solo existe durante la ejecución del método
- ✅ **Útil para dependencias contextuales**
- ⚠️ **Puede complicar la firma del método**

### Cuándo Usar Method Injection

1. **Dependencias que varían por operación** (estrategias, visitantes)
2. **Contexto específico de una llamada** (usuario actual, configuración temporal)
3. **Evitar dependencias permanentes** en la clase
4. **Patrones Strategy o Command**

### Ejemplo Básico

```csharp
public class PaymentProcessor
{
    private readonly IPaymentValidator _validator;
    private readonly ILogger _logger;
    
    public PaymentProcessor(IPaymentValidator validator, ILogger logger)
    {
        _validator = validator ?? throw new ArgumentNullException(nameof(validator));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
    
    // Method Injection: la estrategia de pago se inyecta por método
    public async Task<PaymentResult> ProcessPaymentAsync(
        Payment payment,
        IPaymentStrategy strategy)
    {
        if (strategy == null)
            throw new ArgumentNullException(nameof(strategy));
        
        _logger.Log($"Processing payment with {strategy.GetType().Name}");
        
        if (!_validator.Validate(payment))
        {
            return PaymentResult.Invalid();
        }
        
        // La estrategia específica se usa solo en este método
        return await strategy.ExecuteAsync(payment);
    }
}
```

### Ejemplo Avanzado con Factory Pattern

```csharp
public interface INotificationStrategy
{
    Task SendAsync(string message, string recipient);
}

public class EmailNotificationStrategy : INotificationStrategy
{
    public Task SendAsync(string message, string recipient)
    {
        // Enviar email
        return Task.CompletedTask;
    }
}

public class SmsNotificationStrategy : INotificationStrategy
{
    public Task SendAsync(string message, string recipient)
    {
        // Enviar SMS
        return Task.CompletedTask;
    }
}

public class NotificationService
{
    private readonly ILogger _logger;
    
    public NotificationService(ILogger logger)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
    
    // Method Injection con múltiples estrategias
    public async Task NotifyUserAsync(
        User user,
        string message,
        INotificationStrategy strategy)
    {
        if (user == null) throw new ArgumentNullException(nameof(user));
        if (strategy == null) throw new ArgumentNullException(nameof(strategy));
        
        _logger.Log($"Sending notification to {user.Name}");
        
        await strategy.SendAsync(message, user.ContactInfo);
        
        _logger.Log("Notification sent successfully");
    }
    
    // Método con selección dinámica de estrategia
    public async Task NotifyUserAsync(User user, string message, NotificationType type)
    {
        INotificationStrategy strategy = type switch
        {
            NotificationType.Email => new EmailNotificationStrategy(),
            NotificationType.Sms => new SmsNotificationStrategy(),
            _ => throw new ArgumentException("Invalid notification type")
        };
        
        await NotifyUserAsync(user, message, strategy);
    }
}
```

### Con Factory para Method Injection

```csharp
public interface IPaymentStrategyFactory
{
    IPaymentStrategy GetStrategy(PaymentMethod method);
}

public class PaymentStrategyFactory : IPaymentStrategyFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public PaymentStrategyFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IPaymentStrategy GetStrategy(PaymentMethod method)
    {
        return method switch
        {
            PaymentMethod.CreditCard => _serviceProvider.GetRequiredService<CreditCardStrategy>(),
            PaymentMethod.PayPal => _serviceProvider.GetRequiredService<PayPalStrategy>(),
            PaymentMethod.BankTransfer => _serviceProvider.GetRequiredService<BankTransferStrategy>(),
            _ => throw new NotSupportedException($"Payment method {method} not supported")
        };
    }
}

public class ImprovedPaymentProcessor
{
    private readonly IPaymentStrategyFactory _strategyFactory;
    private readonly ILogger _logger;
    
    public ImprovedPaymentProcessor(
        IPaymentStrategyFactory strategyFactory,
        ILogger logger)
    {
        _strategyFactory = strategyFactory;
        _logger = logger;
    }
    
    public async Task<PaymentResult> ProcessPaymentAsync(Payment payment)
    {
        // Obtener la estrategia dinámicamente
        var strategy = _strategyFactory.GetStrategy(payment.Method);
        
        _logger.Log($"Processing {payment.Method} payment");
        
        return await strategy.ExecuteAsync(payment);
    }
}
```

### Test con Method Injection

```csharp
public class PaymentProcessorTests
{
    [Fact]
    public async Task ProcessPayment_WithCreditCardStrategy_ProcessesSuccessfully()
    {
        // Arrange
        var mockValidator = new Mock<IPaymentValidator>();
        var mockLogger = new Mock<ILogger>();
        var mockStrategy = new Mock<IPaymentStrategy>();
        
        mockValidator.Setup(x => x.Validate(It.IsAny<Payment>())).Returns(true);
        mockStrategy.Setup(x => x.ExecuteAsync(It.IsAny<Payment>()))
            .ReturnsAsync(PaymentResult.Success());
        
        var processor = new PaymentProcessor(mockValidator.Object, mockLogger.Object);
        var payment = new Payment { Amount = 100, Method = PaymentMethod.CreditCard };
        
        // Act
        var result = await processor.ProcessPaymentAsync(payment, mockStrategy.Object);
        
        // Assert
        Assert.True(result.IsSuccessful);
        mockStrategy.Verify(x => x.ExecuteAsync(payment), Times.Once);
    }
    
    [Theory]
    [InlineData(PaymentMethod.CreditCard)]
    [InlineData(PaymentMethod.PayPal)]
    public async Task ProcessPayment_WithDifferentStrategies_UsesCorrectStrategy(
        PaymentMethod method)
    {
        // Arrange
        var mockFactory = new Mock<IPaymentStrategyFactory>();
        var mockLogger = new Mock<ILogger>();
        var mockStrategy = new Mock<IPaymentStrategy>();
        
        mockFactory.Setup(x => x.GetStrategy(method)).Returns(mockStrategy.Object);
        mockStrategy.Setup(x => x.ExecuteAsync(It.IsAny<Payment>()))
            .ReturnsAsync(PaymentResult.Success());
        
        var processor = new ImprovedPaymentProcessor(mockFactory.Object, mockLogger.Object);
        var payment = new Payment { Amount = 100, Method = method };
        
        // Act
        await processor.ProcessPaymentAsync(payment);
        
        // Assert
        mockFactory.Verify(x => x.GetStrategy(method), Times.Once);
        mockStrategy.Verify(x => x.ExecuteAsync(payment), Times.Once);
    }
}
```

---

## Comparación de Patrones

| Aspecto | Constructor Injection | Property Injection | Method Injection |
|---------|----------------------|-------------------|------------------|
| **Obligatoriedad** | ✅ Obligatoria | ⚠️ Opcional | ✅ Obligatoria (por llamada) |
| **Inmutabilidad** | ✅ Inmutable | ❌ Mutable | ✅ Inmutable (scope del método) |
| **Ciclo de vida** | Vida de la instancia | Vida de la instancia | Vida de la llamada |
| **Thread-safety** | ✅ Seguro | ⚠️ Requiere cuidado | ✅ Seguro |
| **Testabilidad** | ✅ Excelente | ✅ Buena | ✅ Excelente |
| **Visibilidad** | ✅ Explícita | ⚠️ Puede ser oculta | ✅ Explícita |
| **Uso recomendado** | Dependencias obligatorias | Dependencias opcionales | Dependencias contextuales |
| **Complejidad** | Baja | Media | Media-Alta |

---

## Ejemplos Prácticos

### Ejemplo 1: Sistema de Autenticación

```csharp
public interface IUserRepository
{
    Task<User?> GetByUsernameAsync(string username);
}

public interface IPasswordHasher
{
    string Hash(string password);
    bool Verify(string password, string hash);
}

public interface ITokenGenerator
{
    string GenerateToken(User user);
}

// Constructor Injection para todas las dependencias obligatorias
public class AuthenticationService
{
    private readonly IUserRepository _userRepository;
    private readonly IPasswordHasher _passwordHasher;
    private readonly ITokenGenerator _tokenGenerator;
    private readonly ILogger _logger;
    
    public AuthenticationService(
        IUserRepository userRepository,
        IPasswordHasher passwordHasher,
        ITokenGenerator tokenGenerator,
        ILogger logger)
    {
        _userRepository = userRepository ?? throw new ArgumentNullException(nameof(userRepository));
        _passwordHasher = passwordHasher ?? throw new ArgumentNullException(nameof(passwordHasher));
        _tokenGenerator = tokenGenerator ?? throw new ArgumentNullException(nameof(tokenGenerator));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
    
    public async Task<AuthenticationResult> AuthenticateAsync(
        string username,
        string password)
    {
        _logger.Log($"Authentication attempt for user: {username}");
        
        var user = await _userRepository.GetByUsernameAsync(username);
        if (user == null)
        {
            _logger.Log($"User not found: {username}");
            return AuthenticationResult.Failed("Invalid credentials");
        }
        
        if (!_passwordHasher.Verify(password, user.PasswordHash))
        {
            _logger.Log($"Invalid password for user: {username}");
            return AuthenticationResult.Failed("Invalid credentials");
        }
        
        var token = _tokenGenerator.GenerateToken(user);
        _logger.Log($"User authenticated successfully: {username}");
        
        return AuthenticationResult.Success(token, user);
    }
}
```

### Ejemplo 2: Sistema de Notificaciones con Estrategias

```csharp
public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public string CustomerEmail { get; set; }
    public NotificationPreference Preference { get; set; }
}

public enum NotificationPreference
{
    Email,
    Sms,
    Both
}

public interface IOrderRepository
{
    Task SaveAsync(Order order);
}

// Constructor Injection + Method Injection
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly ILogger _logger;
    
    public OrderService(IOrderRepository repository, ILogger logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
    
    // Method Injection para la estrategia de notificación
    public async Task ProcessOrderAsync(
        Order order,
        INotificationStrategy notificationStrategy)
    {
        if (order == null) throw new ArgumentNullException(nameof(order));
        if (notificationStrategy == null) 
            throw new ArgumentNullException(nameof(notificationStrategy));
        
        _logger.Log($"Processing order {order.Id}");
        
        await _repository.SaveAsync(order);
        
        await notificationStrategy.SendAsync(
            $"Order {order.Id} confirmed",
            order.CustomerEmail);
        
        _logger.Log($"Order {order.Id} processed successfully");
    }
}
```

### Ejemplo 3: Servicio con Dependencias Opcionales

```csharp
public interface ICacheService
{
    T? Get<T>(string key);
    void Set<T>(string key, T value, TimeSpan expiration);
}

public class ProductService
{
    private readonly IRepository<Product> _repository;
    private readonly ILogger _logger;
    
    // Constructor Injection para dependencias obligatorias
    public ProductService(
        IRepository<Product> repository,
        ILogger logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
    
    // Property Injection para caché (opcional)
    public ICacheService? Cache { get; set; }
    
    public async Task<Product?> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        
        // Usar caché si está disponible
        if (Cache != null)
        {
            var cached = Cache.Get<Product>(cacheKey);
            if (cached != null)
            {
                _logger.Log($"Product {id} retrieved from cache");
                return cached;
            }
        }
        
        _logger.Log($"Fetching product {id} from repository");
        var product = await _repository.GetByIdAsync(id);
        
        // Cachear si el servicio está disponible
        if (product != null && Cache != null)
        {
            Cache.Set(cacheKey, product, TimeSpan.FromMinutes(10));
        }
        
        return product;
    }
}
```

---

## Mejores Prácticas

### 1. Preferir Constructor Injection

```csharp
// ✅ CORRECTO
public class OrderService
{
    private readonly IRepository _repository;
    
    public OrderService(IRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
}

// ❌ EVITAR (sin razón justificada)
public class OrderService
{
    public IRepository Repository { get; set; }
}
```

### 2. Validar Dependencias

```csharp
// ✅ CORRECTO - Validación explícita
public class UserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
}

// ✅ TAMBIÉN CORRECTO - Con C# 11+
public class UserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        ArgumentNullException.ThrowIfNull(repository);
        _repository = repository;
    }
}
```

### 3. Mantener el Constructor Simple

```csharp
// ✅ CORRECTO - Solo asignaciones
public class ProductService
{
    private readonly IRepository _repository;
    private readonly ILogger _logger;
    
    public ProductService(IRepository repository, ILogger logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}

// ❌ EVITAR - Lógica en el constructor
public class ProductService
{
    private readonly IRepository _repository;
    
    public ProductService(IRepository repository)
    {
        _repository = repository;
        _repository.Initialize(); // ❌ No hacer esto
        LoadConfiguration(); // ❌ No hacer esto
    }
}
```

### 4. Usar Interfaces, No Implementaciones Concretas

```csharp
// ✅ CORRECTO
public class OrderService
{
    public OrderService(IEmailService emailService) { }
}

// ❌ EVITAR
public class OrderService
{
    public OrderService(SmtpEmailService emailService) { }
}
```

### 5. Limitar el Número de Dependencias

```csharp
// ⚠️ REVISAR - Demasiadas dependencias (posible violación de SRP)
public class UserService
{
    public UserService(
        IUserRepository userRepository,
        IEmailService emailService,
        ILogger logger,
        IEventBus eventBus,
        INotificationService notificationService,
        IAnalyticsService analyticsService,
        ICacheService cacheService,
        IMetricsService metricsService) // 8+ dependencias
    {
        // Esta clase probablemente hace demasiado
    }
}

// ✅ MEJOR - Dividir responsabilidades
public class UserRegistrationService
{
    public UserRegistrationService(
        IUserRepository userRepository,
        IEmailService emailService,
        IEventBus eventBus) // 3 dependencias relacionadas
    {
    }
}

public class UserNotificationService
{
    public UserNotificationService(
        INotificationService notificationService,
        IAnalyticsService analyticsService) // 2 dependencias relacionadas
    {
    }
}
```

### 6. Property Injection Solo para Opcionales

```csharp
// ✅ CORRECTO - Dependencia opcional con comportamiento predeterminado
public class ReportGenerator
{
    private readonly IDataSource _dataSource;
    private IFormatter _formatter = new DefaultFormatter();
    
    public ReportGenerator(IDataSource dataSource)
    {
        _dataSource = dataSource;
    }
    
    public IFormatter Formatter
    {
        get => _formatter;
        set => _formatter = value ?? throw new ArgumentNullException(nameof(value));
    }
}
```

### 7. Method Injection para Estrategias y Contextos

```csharp
// ✅ CORRECTO - Estrategia varía por operación
public class PaymentProcessor
{
    public Task ProcessAsync(Payment payment, IPaymentStrategy strategy)
    {
        return strategy.ExecuteAsync(payment);
    }
}

// ❌ EVITAR - Usar Method Injection para dependencias constantes
public class UserService
{
    public Task CreateUserAsync(User user, IUserRepository repository) // ❌
    {
        return repository.SaveAsync(user);
    }
}
```

### 8. Documentar Dependencias con XML Comments

```csharp
/// <summary>
/// Service for processing customer orders.
/// </summary>
public class OrderService
{
    /// <summary>
    /// Initializes a new instance of the <see cref="OrderService"/> class.
    /// </summary>
    /// <param name="repository">The repository for persisting orders.</param>
    /// <param name="emailService">The service for sending order confirmations.</param>
    /// <param name="logger">The logger for diagnostic information.</param>
    /// <exception cref="ArgumentNullException">
    /// Thrown when any parameter is null.
    /// </exception>
    public OrderService(
        IOrderRepository repository,
        IEmailService emailService,
        ILogger logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _emailService = emailService ?? throw new ArgumentNullException(nameof(emailService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}
```

---

## Anti-patrones Comunes

### 1. Service Locator (Anti-patrón)

```csharp
// ❌ ANTI-PATRÓN: Service Locator
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        var repository = ServiceLocator.Get<IOrderRepository>(); // ❌
        var emailService = ServiceLocator.Get<IEmailService>(); // ❌
        
        repository.Save(order);
        emailService.Send(order);
    }
}

// ✅ CORRECTO: Constructor Injection
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    
    public OrderService(IOrderRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
    
    public void ProcessOrder(Order order)
    {
        _repository.Save(order);
        _emailService.Send(order);
    }
}
```

**Problemas del Service Locator:**
- Oculta las dependencias
- Dificulta el testing
- Acopla el código al contenedor
- Errores en runtime en lugar de compile-time

### 2. New Keyword en Clases de Negocio

```csharp
// ❌ ANTI-PATRÓN
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        var emailService = new EmailService(); // ❌
        emailService.Send(order);
    }
}

// ✅ CORRECTO
public class OrderService
{
    private readonly IEmailService _emailService;
    
    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public void ProcessOrder(Order order)
    {
        _emailService.Send(order);
    }
}
```

### 3. Temporal Coupling en Property Injection

```csharp
// ❌ ANTI-PATRÓN: Orden de llamadas importa
public class ReportService
{
    public IDataSource DataSource { get; set; }
    
    public string GenerateReport()
    {
        // ❌ NullReferenceException si DataSource no se asignó
        return DataSource.GetData();
    }
}

// Uso problemático
var service = new ReportService();
service.GenerateReport(); // ❌ Boom!
service.DataSource = dataSource; // Demasiado tarde

// ✅ CORRECTO: Constructor Injection elimina el temporal coupling
public class ReportService
{
    private readonly IDataSource _dataSource;
    
    public ReportService(IDataSource dataSource)
    {
        _dataSource = dataSource ?? throw new ArgumentNullException(nameof(dataSource));
    }
    
    public string GenerateReport()
    {
        return _dataSource.GetData(); // Siempre seguro
    }
}
```

### 4. Ambient Context (Anti-patrón)

```csharp
// ❌ ANTI-PATRÓN: Contexto ambiental
public static class CurrentUser
{
    public static User User { get; set; } // ❌ Estado global
}

public class OrderService
{
    public void ProcessOrder(Order order)
    {
        var user = CurrentUser.User; // ❌ Dependencia oculta
        // ...
    }
}

// ✅ CORRECTO: Inyección explícita
public interface IUserContext
{
    User CurrentUser { get; }
}

public class OrderService
{
    private readonly IUserContext _userContext;
    
    public OrderService(IUserContext userContext)
    {
        _userContext = userContext;
    }
    
    public void ProcessOrder(Order order)
    {
        var user = _userContext.CurrentUser; // ✅ Explícito y testeable
        // ...
    }
}
```

### 5. Over-injection

```csharp
// ❌ ANTI-PATRÓN: Inyectar IServiceProvider
public class OrderService
{
    private readonly IServiceProvider _serviceProvider;
    
    public OrderService(IServiceProvider serviceProvider) // ❌
    {
        _serviceProvider = serviceProvider;
    }
    
    public void ProcessOrder(Order order)
    {
        var repository = _serviceProvider.GetService<IOrderRepository>();
        // Esto es básicamente Service Locator
    }
}

// ✅ CORRECTO: Inyectar dependencias específicas
public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }
}
```

### 6. Control Freak (Control Excesivo)

```csharp
// ❌ ANTI-PATRÓN: La clase controla demasiado
public class UserService
{
    private IUserRepository _repository;
    
    public void SetRepository(IUserRepository repository)
    {
        if (repository == null)
            _repository = new DefaultUserRepository(); // ❌ Control excesivo
        else
            _repository = repository;
    }
}

// ✅ CORRECTO: Confiar en el contenedor DI
public class UserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
}
```

---

## Conclusión

La Inyección de Dependencias es fundamental para crear código testeable y mantenible en .NET. Los tres patrones principales tienen sus usos específicos:

- **Constructor Injection**: El patrón predeterminado para dependencias obligatorias
- **Property Injection**: Para dependencias opcionales con valores predeterminados
- **Method Injection**: Para estrategias y contextos que varían por operación

Al combinar DI con los principios SOLID, especialmente el principio de Inversión de Dependencias, se crea código desacoplado, fácil de testear y de mantener a largo plazo.

**Regla de oro**: Cuando tengas dudas, usa Constructor Injection. Es el patrón más simple, seguro y explícito.

---

## Recursos Adicionales

- [Microsoft Docs: Dependency Injection in .NET](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)
- [Dependency Injection Principles, Practices, and Patterns](https://www.manning.com/books/dependency-injection-principles-practices-patterns) - Steven van Deursen & Mark Seemann
- [Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164) - Robert C. Martin

---



# Patrones de Diseño que Facilitan el Testing

## Introducción

Los patrones de diseño son soluciones probadas a problemas comunes en el desarrollo de software. Cuando se aplican correctamente, no solo mejoran la estructura y mantenibilidad del código, sino que también facilitan enormemente la creación de pruebas efectivas. En el contexto de .NET Framework, estos patrones son fundamentales para construir aplicaciones testeables y robustas.

Este documento explora los patrones de diseño más relevantes que mejoran la testeabilidad del código, con ejemplos prácticos en C# para .NET Framework.

---

## 1. Patrón Repository

### Descripción

El patrón Repository actúa como una capa de abstracción entre la lógica de negocio y el acceso a datos. Encapsula la lógica necesaria para acceder a las fuentes de datos, proporcionando una interfaz más orientada a objetos.

### Ventajas para Testing

- Permite reemplazar fácilmente el acceso real a datos con implementaciones mock
- Aísla la lógica de negocio de los detalles de persistencia
- Facilita las pruebas unitarias sin necesidad de bases de datos reales

### Ejemplo en .NET Framework

```csharp
// Definición de la interfaz del repositorio
public interface IProductRepository
{
    Product GetById(int id);
    IEnumerable<Product> GetAll();
    void Add(Product product);
    void Update(Product product);
    void Delete(int id);
}

// Implementación real del repositorio
public class ProductRepository : IProductRepository
{
    private readonly string _connectionString;

    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public Product GetById(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var command = new SqlCommand(
                "SELECT * FROM Products WHERE Id = @Id", connection);
            command.Parameters.AddWithValue("@Id", id);
            
            using (var reader = command.ExecuteReader())
            {
                if (reader.Read())
                {
                    return new Product
                    {
                        Id = (int)reader["Id"],
                        Name = reader["Name"].ToString(),
                        Price = (decimal)reader["Price"]
                    };
                }
            }
        }
        return null;
    }

    // Implementación de otros métodos...
}

// Servicio que usa el repositorio
public class ProductService
{
    private readonly IProductRepository _repository;

    public ProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public decimal GetProductPrice(int productId)
    {
        var product = _repository.GetById(productId);
        if (product == null)
        {
            throw new ArgumentException("Producto no encontrado");
        }
        return product.Price;
    }
}
```

### Prueba Unitaria con Mock

```csharp
[TestClass]
public class ProductServiceTests
{
    [TestMethod]
    public void GetProductPrice_ProductExists_ReturnsCorrectPrice()
    {
        // Arrange
        var mockRepository = new Mock<IProductRepository>();
        var expectedProduct = new Product { Id = 1, Name = "Test", Price = 99.99m };
        
        mockRepository.Setup(r => r.GetById(1)).Returns(expectedProduct);
        
        var service = new ProductService(mockRepository.Object);

        // Act
        var result = service.GetProductPrice(1);

        // Assert
        Assert.AreEqual(99.99m, result);
        mockRepository.Verify(r => r.GetById(1), Times.Once);
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void GetProductPrice_ProductNotFound_ThrowsException()
    {
        // Arrange
        var mockRepository = new Mock<IProductRepository>();
        mockRepository.Setup(r => r.GetById(It.IsAny<int>())).Returns((Product)null);
        
        var service = new ProductService(mockRepository.Object);

        // Act
        service.GetProductPrice(999);
    }
}
```

---

## 2. Patrón Strategy

### Descripción

El patrón Strategy define una familia de algoritmos, encapsula cada uno de ellos y los hace intercambiables. Permite que el algoritmo varíe independientemente de los clientes que lo utilizan.

### Ventajas para Testing

- Cada estrategia se puede probar de forma aislada
- Facilita el testing de diferentes comportamientos sin modificar el contexto
- Simplifica la creación de mocks para diferentes estrategias

### Ejemplo en .NET Framework

```csharp
// Interfaz de la estrategia
public interface IShippingStrategy
{
    decimal CalculateShippingCost(decimal orderTotal, string destination);
}

// Estrategia concreta: Envío estándar
public class StandardShippingStrategy : IShippingStrategy
{
    public decimal CalculateShippingCost(decimal orderTotal, string destination)
    {
        if (orderTotal > 100)
            return 0; // Envío gratis para pedidos mayores a 100
        return 10m;
    }
}

// Estrategia concreta: Envío express
public class ExpressShippingStrategy : IShippingStrategy
{
    public decimal CalculateShippingCost(decimal orderTotal, string destination)
    {
        return destination.StartsWith("Madrid") ? 15m : 25m;
    }
}

// Contexto que usa las estrategias
public class OrderProcessor
{
    private readonly IShippingStrategy _shippingStrategy;

    public OrderProcessor(IShippingStrategy shippingStrategy)
    {
        _shippingStrategy = shippingStrategy;
    }

    public decimal CalculateTotalCost(decimal orderTotal, string destination)
    {
        var shippingCost = _shippingStrategy.CalculateShippingCost(
            orderTotal, destination);
        return orderTotal + shippingCost;
    }
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class OrderProcessorTests
{
    [TestMethod]
    public void CalculateTotalCost_WithStandardShipping_AddsFreeShipping()
    {
        // Arrange
        var strategy = new StandardShippingStrategy();
        var processor = new OrderProcessor(strategy);

        // Act
        var result = processor.CalculateTotalCost(150m, "Barcelona");

        // Assert
        Assert.AreEqual(150m, result); // Sin coste de envío
    }

    [TestMethod]
    public void CalculateTotalCost_WithExpressShipping_AddsCorrectCost()
    {
        // Arrange
        var strategy = new ExpressShippingStrategy();
        var processor = new OrderProcessor(strategy);

        // Act
        var result = processor.CalculateTotalCost(100m, "Madrid");

        // Assert
        Assert.AreEqual(115m, result); // 100 + 15
    }

    [TestMethod]
    public void CalculateTotalCost_WithMockedStrategy_UsesStrategy()
    {
        // Arrange
        var mockStrategy = new Mock<IShippingStrategy>();
        mockStrategy.Setup(s => s.CalculateShippingCost(100m, "Test"))
                   .Returns(5m);
        
        var processor = new OrderProcessor(mockStrategy.Object);

        // Act
        var result = processor.CalculateTotalCost(100m, "Test");

        // Assert
        Assert.AreEqual(105m, result);
        mockStrategy.Verify(s => s.CalculateShippingCost(100m, "Test"), Times.Once);
    }
}
```

---

## 3. Patrón Factory

### Descripción

El patrón Factory proporciona una interfaz para crear objetos en una superclase, pero permite a las subclases alterar el tipo de objetos que se crearán.

### Ventajas para Testing

- Facilita la inyección de factories mock para controlar la creación de objetos
- Permite testear la lógica de creación de forma aislada
- Simplifica la configuración de escenarios de prueba complejos

### Ejemplo en .NET Framework

```csharp
// Interfaz del producto
public interface IEmailSender
{
    void SendEmail(string to, string subject, string body);
}

// Implementaciones concretas
public class SmtpEmailSender : IEmailSender
{
    private readonly string _smtpServer;

    public SmtpEmailSender(string smtpServer)
    {
        _smtpServer = smtpServer;
    }

    public void SendEmail(string to, string subject, string body)
    {
        // Lógica real de envío SMTP
        var client = new SmtpClient(_smtpServer);
        var message = new MailMessage("noreply@empresa.com", to, subject, body);
        client.Send(message);
    }
}

public class SendGridEmailSender : IEmailSender
{
    private readonly string _apiKey;

    public SendGridEmailSender(string apiKey)
    {
        _apiKey = apiKey;
    }

    public void SendEmail(string to, string subject, string body)
    {
        // Lógica de envío usando SendGrid API
    }
}

// Factory
public interface IEmailSenderFactory
{
    IEmailSender CreateEmailSender();
}

public class EmailSenderFactory : IEmailSenderFactory
{
    private readonly string _provider;
    private readonly string _configuration;

    public EmailSenderFactory(string provider, string configuration)
    {
        _provider = provider;
        _configuration = configuration;
    }

    public IEmailSender CreateEmailSender()
    {
        switch (_provider.ToLower())
        {
            case "smtp":
                return new SmtpEmailSender(_configuration);
            case "sendgrid":
                return new SendGridEmailSender(_configuration);
            default:
                throw new ArgumentException($"Proveedor desconocido: {_provider}");
        }
    }
}

// Servicio que usa la factory
public class NotificationService
{
    private readonly IEmailSenderFactory _emailSenderFactory;

    public NotificationService(IEmailSenderFactory emailSenderFactory)
    {
        _emailSenderFactory = emailSenderFactory;
    }

    public void SendWelcomeEmail(string userEmail)
    {
        var emailSender = _emailSenderFactory.CreateEmailSender();
        emailSender.SendEmail(
            userEmail,
            "Bienvenido",
            "Gracias por registrarte en nuestra aplicación");
    }
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class NotificationServiceTests
{
    [TestMethod]
    public void SendWelcomeEmail_CallsEmailSender()
    {
        // Arrange
        var mockEmailSender = new Mock<IEmailSender>();
        var mockFactory = new Mock<IEmailSenderFactory>();
        
        mockFactory.Setup(f => f.CreateEmailSender())
                  .Returns(mockEmailSender.Object);
        
        var service = new NotificationService(mockFactory.Object);

        // Act
        service.SendWelcomeEmail("test@example.com");

        // Assert
        mockEmailSender.Verify(
            s => s.SendEmail(
                "test@example.com",
                "Bienvenido",
                It.IsAny<string>()),
            Times.Once);
    }
}

[TestClass]
public class EmailSenderFactoryTests
{
    [TestMethod]
    public void CreateEmailSender_SmtpProvider_ReturnsSmtpEmailSender()
    {
        // Arrange
        var factory = new EmailSenderFactory("smtp", "smtp.empresa.com");

        // Act
        var result = factory.CreateEmailSender();

        // Assert
        Assert.IsInstanceOfType(result, typeof(SmtpEmailSender));
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void CreateEmailSender_InvalidProvider_ThrowsException()
    {
        // Arrange
        var factory = new EmailSenderFactory("invalid", "config");

        // Act
        factory.CreateEmailSender();
    }
}
```

---

## 4. Patrón Adapter

### Descripción

El patrón Adapter permite que interfaces incompatibles trabajen juntas. Convierte la interfaz de una clase en otra interfaz que el cliente espera.

### Ventajas para Testing

- Permite crear wrappers testeables alrededor de código legacy o APIs externas
- Facilita el aislamiento de dependencias externas difíciles de testear
- Simplifica la creación de mocks para sistemas externos

### Ejemplo en .NET Framework

```csharp
// Interfaz que queremos usar en nuestra aplicación
public interface IPaymentGateway
{
    PaymentResult ProcessPayment(decimal amount, string cardNumber);
}

// Resultado estándar
public class PaymentResult
{
    public bool Success { get; set; }
    public string TransactionId { get; set; }
    public string ErrorMessage { get; set; }
}

// API externa de terceros (que no podemos modificar)
public class ExternalPaymentAPI
{
    public string MakePayment(string amount, string card, string currency)
    {
        // Simulación de llamada a API externa
        // Retorna XML: "<Response><Status>OK</Status><Id>12345</Id></Response>"
        return "<Response><Status>OK</Status><Id>12345</Id></Response>";
    }
}

// Adapter que adapta la API externa a nuestra interfaz
public class PaymentGatewayAdapter : IPaymentGateway
{
    private readonly ExternalPaymentAPI _externalApi;

    public PaymentGatewayAdapter(ExternalPaymentAPI externalApi)
    {
        _externalApi = externalApi;
    }

    public PaymentResult ProcessPayment(decimal amount, string cardNumber)
    {
        try
        {
            // Convertir y llamar a la API externa
            var xmlResponse = _externalApi.MakePayment(
                amount.ToString("F2"),
                cardNumber,
                "EUR");

            // Parsear la respuesta XML
            var xmlDoc = new System.Xml.XmlDocument();
            xmlDoc.LoadXml(xmlResponse);

            var status = xmlDoc.SelectSingleNode("//Status")?.InnerText;
            var id = xmlDoc.SelectSingleNode("//Id")?.InnerText;

            return new PaymentResult
            {
                Success = status == "OK",
                TransactionId = id,
                ErrorMessage = status == "OK" ? null : "Error en el pago"
            };
        }
        catch (Exception ex)
        {
            return new PaymentResult
            {
                Success = false,
                ErrorMessage = ex.Message
            };
        }
    }
}

// Servicio que usa el adapter
public class CheckoutService
{
    private readonly IPaymentGateway _paymentGateway;

    public CheckoutService(IPaymentGateway paymentGateway)
    {
        _paymentGateway = paymentGateway;
    }

    public bool CompleteOrder(decimal orderTotal, string cardNumber)
    {
        var result = _paymentGateway.ProcessPayment(orderTotal, cardNumber);
        
        if (result.Success)
        {
            // Guardar pedido en base de datos
            return true;
        }
        
        return false;
    }
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class CheckoutServiceTests
{
    [TestMethod]
    public void CompleteOrder_SuccessfulPayment_ReturnsTrue()
    {
        // Arrange
        var mockGateway = new Mock<IPaymentGateway>();
        mockGateway.Setup(g => g.ProcessPayment(100m, "1234"))
                  .Returns(new PaymentResult 
                  { 
                      Success = true, 
                      TransactionId = "TX123" 
                  });

        var service = new CheckoutService(mockGateway.Object);

        // Act
        var result = service.CompleteOrder(100m, "1234");

        // Assert
        Assert.IsTrue(result);
        mockGateway.Verify(g => g.ProcessPayment(100m, "1234"), Times.Once);
    }

    [TestMethod]
    public void CompleteOrder_FailedPayment_ReturnsFalse()
    {
        // Arrange
        var mockGateway = new Mock<IPaymentGateway>();
        mockGateway.Setup(g => g.ProcessPayment(It.IsAny<decimal>(), It.IsAny<string>()))
                  .Returns(new PaymentResult 
                  { 
                      Success = false, 
                      ErrorMessage = "Fondos insuficientes" 
                  });

        var service = new CheckoutService(mockGateway.Object);

        // Act
        var result = service.CompleteOrder(100m, "1234");

        // Assert
        Assert.IsFalse(result);
    }
}
```

---

## 5. Patrón Observer

### Descripción

El patrón Observer define una dependencia uno-a-muchos entre objetos, de manera que cuando un objeto cambia de estado, todos sus dependientes son notificados y actualizados automáticamente.

### Ventajas para Testing

- Permite probar la lógica de notificación de forma aislada
- Facilita verificar que los observadores son notificados correctamente
- Simplifica el testing de eventos y reacciones en cadena

### Ejemplo en .NET Framework

```csharp
// Interfaz del observador
public interface IOrderObserver
{
    void OnOrderCreated(Order order);
    void OnOrderStatusChanged(Order order, string oldStatus, string newStatus);
}

// Sujeto observable
public class OrderManager
{
    private readonly List<IOrderObserver> _observers;
    private readonly List<Order> _orders;

    public OrderManager()
    {
        _observers = new List<IOrderObserver>();
        _orders = new List<Order>();
    }

    public void RegisterObserver(IOrderObserver observer)
    {
        if (!_observers.Contains(observer))
        {
            _observers.Add(observer);
        }
    }

    public void UnregisterObserver(IOrderObserver observer)
    {
        _observers.Remove(observer);
    }

    public void CreateOrder(Order order)
    {
        _orders.Add(order);
        NotifyOrderCreated(order);
    }

    public void UpdateOrderStatus(int orderId, string newStatus)
    {
        var order = _orders.FirstOrDefault(o => o.Id == orderId);
        if (order != null)
        {
            var oldStatus = order.Status;
            order.Status = newStatus;
            NotifyOrderStatusChanged(order, oldStatus, newStatus);
        }
    }

    private void NotifyOrderCreated(Order order)
    {
        foreach (var observer in _observers)
        {
            observer.OnOrderCreated(order);
        }
    }

    private void NotifyOrderStatusChanged(Order order, string oldStatus, string newStatus)
    {
        foreach (var observer in _observers)
        {
            observer.OnOrderStatusChanged(order, oldStatus, newStatus);
        }
    }
}

// Observador concreto: Logger
public class OrderLogger : IOrderObserver
{
    private readonly ILogger _logger;

    public OrderLogger(ILogger logger)
    {
        _logger = logger;
    }

    public void OnOrderCreated(Order order)
    {
        _logger.Log($"Pedido creado: {order.Id} - Total: {order.Total}");
    }

    public void OnOrderStatusChanged(Order order, string oldStatus, string newStatus)
    {
        _logger.Log($"Pedido {order.Id} cambió de {oldStatus} a {newStatus}");
    }
}

// Observador concreto: Email Notifier
public class OrderEmailNotifier : IOrderObserver
{
    private readonly IEmailSender _emailSender;

    public OrderEmailNotifier(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }

    public void OnOrderCreated(Order order)
    {
        _emailSender.SendEmail(
            order.CustomerEmail,
            "Pedido confirmado",
            $"Tu pedido #{order.Id} ha sido confirmado");
    }

    public void OnOrderStatusChanged(Order order, string oldStatus, string newStatus)
    {
        if (newStatus == "Enviado")
        {
            _emailSender.SendEmail(
                order.CustomerEmail,
                "Pedido enviado",
                $"Tu pedido #{order.Id} ha sido enviado");
        }
    }
}

// Modelo
public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public string CustomerEmail { get; set; }
    public string Status { get; set; }
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class OrderManagerTests
{
    [TestMethod]
    public void CreateOrder_NotifiesAllObservers()
    {
        // Arrange
        var manager = new OrderManager();
        var observer1 = new Mock<IOrderObserver>();
        var observer2 = new Mock<IOrderObserver>();
        
        manager.RegisterObserver(observer1.Object);
        manager.RegisterObserver(observer2.Object);

        var order = new Order { Id = 1, Total = 100m };

        // Act
        manager.CreateOrder(order);

        // Assert
        observer1.Verify(o => o.OnOrderCreated(order), Times.Once);
        observer2.Verify(o => o.OnOrderCreated(order), Times.Once);
    }

    [TestMethod]
    public void UpdateOrderStatus_NotifiesStatusChange()
    {
        // Arrange
        var manager = new OrderManager();
        var observer = new Mock<IOrderObserver>();
        manager.RegisterObserver(observer.Object);

        var order = new Order { Id = 1, Status = "Pendiente" };
        manager.CreateOrder(order);

        // Act
        manager.UpdateOrderStatus(1, "Enviado");

        // Assert
        observer.Verify(
            o => o.OnOrderStatusChanged(
                It.Is<Order>(ord => ord.Id == 1),
                "Pendiente",
                "Enviado"),
            Times.Once);
    }

    [TestMethod]
    public void UnregisterObserver_StopsNotifications()
    {
        // Arrange
        var manager = new OrderManager();
        var observer = new Mock<IOrderObserver>();
        
        manager.RegisterObserver(observer.Object);
        manager.UnregisterObserver(observer.Object);

        var order = new Order { Id = 1 };

        // Act
        manager.CreateOrder(order);

        // Assert
        observer.Verify(o => o.OnOrderCreated(It.IsAny<Order>()), Times.Never);
    }
}

[TestClass]
public class OrderEmailNotifierTests
{
    [TestMethod]
    public void OnOrderCreated_SendsConfirmationEmail()
    {
        // Arrange
        var mockEmailSender = new Mock<IEmailSender>();
        var notifier = new OrderEmailNotifier(mockEmailSender.Object);
        
        var order = new Order 
        { 
            Id = 1, 
            CustomerEmail = "customer@example.com" 
        };

        // Act
        notifier.OnOrderCreated(order);

        // Assert
        mockEmailSender.Verify(
            e => e.SendEmail(
                "customer@example.com",
                "Pedido confirmado",
                It.IsAny<string>()),
            Times.Once);
    }

    [TestMethod]
    public void OnOrderStatusChanged_ToShipped_SendsShippingEmail()
    {
        // Arrange
        var mockEmailSender = new Mock<IEmailSender>();
        var notifier = new OrderEmailNotifier(mockEmailSender.Object);
        
        var order = new Order 
        { 
            Id = 1, 
            CustomerEmail = "customer@example.com" 
        };

        // Act
        notifier.OnOrderStatusChanged(order, "Pendiente", "Enviado");

        // Assert
        mockEmailSender.Verify(
            e => e.SendEmail(
                "customer@example.com",
                "Pedido enviado",
                It.IsAny<string>()),
            Times.Once);
    }
}
```

---

## 6. Patrón Decorator

### Descripción

El patrón Decorator permite añadir funcionalidades adicionales a un objeto dinámicamente. Proporciona una alternativa flexible a la herencia para extender funcionalidad.

### Ventajas para Testing

- Permite probar cada decorador de forma independiente
- Facilita la composición de comportamientos para pruebas específicas
- Simplifica el testing de funcionalidades transversales (logging, caching, validación)

### Ejemplo en .NET Framework

```csharp
// Interfaz base
public interface IDataService
{
    string GetData(int id);
    void SaveData(int id, string data);
}

// Implementación base
public class DataService : IDataService
{
    public string GetData(int id)
    {
        // Simulación de acceso a base de datos
        return $"Data_{id}";
    }

    public void SaveData(int id, string data)
    {
        // Simulación de guardado
    }
}

// Decorador base abstracto
public abstract class DataServiceDecorator : IDataService
{
    protected readonly IDataService _innerService;

    protected DataServiceDecorator(IDataService innerService)
    {
        _innerService = innerService;
    }

    public virtual string GetData(int id)
    {
        return _innerService.GetData(id);
    }

    public virtual void SaveData(int id, string data)
    {
        _innerService.SaveData(id, data);
    }
}

// Decorador: Logging
public class LoggingDataServiceDecorator : DataServiceDecorator
{
    private readonly ILogger _logger;

    public LoggingDataServiceDecorator(IDataService innerService, ILogger logger)
        : base(innerService)
    {
        _logger = logger;
    }

    public override string GetData(int id)
    {
        _logger.Log($"Obteniendo datos para ID: {id}");
        var result = base.GetData(id);
        _logger.Log($"Datos obtenidos: {result}");
        return result;
    }

    public override void SaveData(int id, string data)
    {
        _logger.Log($"Guardando datos para ID: {id}");
        base.SaveData(id, data);
        _logger.Log($"Datos guardados exitosamente");
    }
}

// Decorador: Caching
public class CachingDataServiceDecorator : DataServiceDecorator
{
    private readonly Dictionary<int, string> _cache;

    public CachingDataServiceDecorator(IDataService innerService)
        : base(innerService)
    {
        _cache = new Dictionary<int, string>();
    }

    public override string GetData(int id)
    {
        if (_cache.ContainsKey(id))
        {
            return _cache[id];
        }

        var result = base.GetData(id);
        _cache[id] = result;
        return result;
    }

    public override void SaveData(int id, string data)
    {
        base.SaveData(id, data);
        _cache[id] = data; // Actualizar caché
    }
}

// Decorador: Validación
public class ValidatingDataServiceDecorator : DataServiceDecorator
{
    public ValidatingDataServiceDecorator(IDataService innerService)
        : base(innerService)
    {
    }

    public override string GetData(int id)
    {
        if (id <= 0)
        {
            throw new ArgumentException("ID debe ser mayor que cero");
        }
        return base.GetData(id);
    }

    public override void SaveData(int id, string data)
    {
        if (id <= 0)
        {
            throw new ArgumentException("ID debe ser mayor que cero");
        }
        if (string.IsNullOrWhiteSpace(data))
        {
            throw new ArgumentException("Los datos no pueden estar vacíos");
        }
        base.SaveData(id, data);
    }
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class DataServiceDecoratorTests
{
    [TestMethod]
    public void LoggingDecorator_LogsGetDataCall()
    {
        // Arrange
        var mockInnerService = new Mock<IDataService>();
        var mockLogger = new Mock<ILogger>();
        
        mockInnerService.Setup(s => s.GetData(1)).Returns("TestData");
        
        var decorator = new LoggingDataServiceDecorator(
            mockInnerService.Object,
            mockLogger.Object);

        // Act
        var result = decorator.GetData(1);

        // Assert
        Assert.AreEqual("TestData", result);
        mockLogger.Verify(l => l.Log(It.IsAny<string>()), Times.AtLeast(2));
        mockInnerService.Verify(s => s.GetData(1), Times.Once);
    }

    [TestMethod]
    public void CachingDecorator_CachesResults()
    {
        // Arrange
        var mockInnerService = new Mock<IDataService>();
        mockInnerService.Setup(s => s.GetData(1)).Returns("TestData");
        
        var decorator = new CachingDataServiceDecorator(mockInnerService.Object);

        // Act
        var result1 = decorator.GetData(1);
        var result2 = decorator.GetData(1);

        // Assert
        Assert.AreEqual("TestData", result1);
        Assert.AreEqual("TestData", result2);
        // Debe llamarse solo una vez porque la segunda está en caché
        mockInnerService.Verify(s => s.GetData(1), Times.Once);
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void ValidatingDecorator_ThrowsOnInvalidId()
    {
        // Arrange
        var mockInnerService = new Mock<IDataService>();
        var decorator = new ValidatingDataServiceDecorator(mockInnerService.Object);

        // Act
        decorator.GetData(-1);
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void ValidatingDecorator_ThrowsOnEmptyData()
    {
        // Arrange
        var mockInnerService = new Mock<IDataService>();
        var decorator = new ValidatingDataServiceDecorator(mockInnerService.Object);

        // Act
        decorator.SaveData(1, "");
    }

    [TestMethod]
    public void MultipleDecorators_WorkTogether()
    {
        // Arrange
        var mockLogger = new Mock<ILogger>();
        var baseService = new DataService();
        
        // Composición de decoradores
        IDataService service = baseService;
        service = new ValidatingDataServiceDecorator(service);
        service = new CachingDataServiceDecorator(service);
        service = new LoggingDataServiceDecorator(service, mockLogger.Object);

        // Act
        var result = service.GetData(1);

        // Assert
        Assert.IsNotNull(result);
        mockLogger.Verify(l => l.Log(It.IsAny<string>()), Times.AtLeast(2));
    }
}
```

---

## 7. Patrón Template Method

### Descripción

El patrón Template Method define el esqueleto de un algoritmo en una clase base, permitiendo que las subclases redefinan ciertos pasos sin cambiar la estructura general.

### Ventajas para Testing

- Permite probar la estructura del algoritmo de forma aislada
- Facilita el testing de pasos específicos mediante herencia o mocking
- Simplifica la verificación de la secuencia de ejecución

### Ejemplo en .NET Framework

```csharp
// Clase abstracta con el template method
public abstract class DataImporter
{
    // Template Method - define la estructura del algoritmo
    public ImportResult Import(string filePath)
    {
        var result = new ImportResult();

        try
        {
            // Paso 1: Validar
            if (!ValidateFile(filePath))
            {
                result.Success = false;
                result.ErrorMessage = "Archivo no válido";
                return result;
            }

            // Paso 2: Leer datos
            var data = ReadData(filePath);

            // Paso 3: Transformar (opcional - hook)
            data = TransformData(data);

            // Paso 4: Validar datos
            if (!ValidateData(data))
            {
                result.Success = false;
                result.ErrorMessage = "Datos no válidos";
                return result;
            }

            // Paso 5: Guardar
            SaveData(data);

            // Paso 6: Limpieza
            Cleanup();

            result.Success = true;
            result.RecordsImported = data.Count;
        }
        catch (Exception ex)
        {
            result.Success = false;
            result.ErrorMessage = ex.Message;
        }

        return result;
    }

    // Métodos abstractos que deben implementar las subclases
    protected abstract bool ValidateFile(string filePath);
    protected abstract List<string> ReadData(string filePath);
    protected abstract void SaveData(List<string> data);

    // Hook method - implementación por defecto que puede ser sobrescrita
    protected virtual List<string> TransformData(List<string> data)
    {
        return data; // Sin transformación por defecto
    }

    // Método concreto con validación básica
    protected virtual bool ValidateData(List<string> data)
    {
        return data != null && data.Count > 0;
    }

    // Hook para limpieza
    protected virtual void Cleanup()
    {
        // Sin limpieza por defecto
    }
}

// Implementación concreta: CSV Importer
public class CsvDataImporter : DataImporter
{
    private readonly IFileSystem _fileSystem;
    private readonly IDataRepository _repository;

    public CsvDataImporter(IFileSystem fileSystem, IDataRepository repository)
    {
        _fileSystem = fileSystem;
        _repository = repository;
    }

    protected override bool ValidateFile(string filePath)
    {
        return _fileSystem.FileExists(filePath) && 
               filePath.EndsWith(".csv", StringComparison.OrdinalIgnoreCase);
    }

    protected override List<string> ReadData(string filePath)
    {
        var lines = _fileSystem.ReadAllLines(filePath);
        return new List<string>(lines);
    }

    protected override List<string> TransformData(List<string> data)
    {
        // Eliminar encabezados CSV
        if (data.Count > 0)
        {
            data.RemoveAt(0);
        }
        return data;
    }

    protected override void SaveData(List<string> data)
    {
        foreach (var line in data)
        {
            var fields = line.Split(',');
            _repository.Insert(fields);
        }
    }
}

// Implementación concreta: XML Importer
public class XmlDataImporter : DataImporter
{
    private readonly IFileSystem _fileSystem;
    private readonly IDataRepository _repository;

    public XmlDataImporter(IFileSystem fileSystem, IDataRepository repository)
    {
        _fileSystem = fileSystem;
        _repository = repository;
    }

    protected override bool ValidateFile(string filePath)
    {
        return _fileSystem.FileExists(filePath) && 
               filePath.EndsWith(".xml", StringComparison.OrdinalIgnoreCase);
    }

    protected override List<string> ReadData(string filePath)
    {
        var xmlContent = _fileSystem.ReadAllText(filePath);
        var xmlDoc = new System.Xml.XmlDocument();
        xmlDoc.LoadXml(xmlContent);

        var nodes = xmlDoc.SelectNodes("//Record");
        var data = new List<string>();
        
        foreach (System.Xml.XmlNode node in nodes)
        {
            data.Add(node.InnerText);
        }

        return data;
    }

    protected override void SaveData(List<string> data)
    {
        foreach (var record in data)
        {
            _repository.Insert(new[] { record });
        }
    }

    protected override void Cleanup()
    {
        // Mover archivo procesado a carpeta de respaldo
        base.Cleanup();
    }
}

// Modelos de apoyo
public class ImportResult
{
    public bool Success { get; set; }
    public int RecordsImported { get; set; }
    public string ErrorMessage { get; set; }
}

public interface IFileSystem
{
    bool FileExists(string path);
    string[] ReadAllLines(string path);
    string ReadAllText(string path);
}

public interface IDataRepository
{
    void Insert(string[] fields);
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class CsvDataImporterTests
{
    [TestMethod]
    public void Import_ValidCsvFile_ReturnsSuccess()
    {
        // Arrange
        var mockFileSystem = new Mock<IFileSystem>();
        var mockRepository = new Mock<IDataRepository>();

        mockFileSystem.Setup(fs => fs.FileExists("test.csv")).Returns(true);
        mockFileSystem.Setup(fs => fs.ReadAllLines("test.csv"))
                     .Returns(new[] { "Header1,Header2", "Value1,Value2", "Value3,Value4" });

        var importer = new CsvDataImporter(
            mockFileSystem.Object,
            mockRepository.Object);

        // Act
        var result = importer.Import("test.csv");

        // Assert
        Assert.IsTrue(result.Success);
        Assert.AreEqual(2, result.RecordsImported);
        mockRepository.Verify(r => r.Insert(It.IsAny<string[]>()), Times.Exactly(2));
    }

    [TestMethod]
    public void Import_FileNotExists_ReturnsFailure()
    {
        // Arrange
        var mockFileSystem = new Mock<IFileSystem>();
        var mockRepository = new Mock<IDataRepository>();

        mockFileSystem.Setup(fs => fs.FileExists(It.IsAny<string>())).Returns(false);

        var importer = new CsvDataImporter(
            mockFileSystem.Object,
            mockRepository.Object);

        // Act
        var result = importer.Import("nonexistent.csv");

        // Assert
        Assert.IsFalse(result.Success);
        Assert.IsNotNull(result.ErrorMessage);
        mockRepository.Verify(r => r.Insert(It.IsAny<string[]>()), Times.Never);
    }

    [TestMethod]
    public void Import_WrongExtension_ReturnsFailure()
    {
        // Arrange
        var mockFileSystem = new Mock<IFileSystem>();
        var mockRepository = new Mock<IDataRepository>();

        mockFileSystem.Setup(fs => fs.FileExists("test.txt")).Returns(true);

        var importer = new CsvDataImporter(
            mockFileSystem.Object,
            mockRepository.Object);

        // Act
        var result = importer.Import("test.txt");

        // Assert
        Assert.IsFalse(result.Success);
    }
}

// Clase de prueba personalizada para verificar el Template Method
[TestClass]
public class TestableDataImporter : DataImporter
{
    public bool ValidateFileCalled { get; private set; }
    public bool ReadDataCalled { get; private set; }
    public bool TransformDataCalled { get; private set; }
    public bool SaveDataCalled { get; private set; }

    protected override bool ValidateFile(string filePath)
    {
        ValidateFileCalled = true;
        return true;
    }

    protected override List<string> ReadData(string filePath)
    {
        ReadDataCalled = true;
        return new List<string> { "data1", "data2" };
    }

    protected override List<string> TransformData(List<string> data)
    {
        TransformDataCalled = true;
        return base.TransformData(data);
    }

    protected override void SaveData(List<string> data)
    {
        SaveDataCalled = true;
    }
}

[TestClass]
public class TemplateMethodTests
{
    [TestMethod]
    public void Import_CallsAllMethodsInCorrectOrder()
    {
        // Arrange
        var importer = new TestableDataImporter();

        // Act
        var result = importer.Import("test.txt");

        // Assert
        Assert.IsTrue(importer.ValidateFileCalled);
        Assert.IsTrue(importer.ReadDataCalled);
        Assert.IsTrue(importer.TransformDataCalled);
        Assert.IsTrue(importer.SaveDataCalled);
        Assert.IsTrue(result.Success);
    }
}
```

---

## 8. Patrón Model-View-Presenter (MVP) para WebForms

### Descripción

El patrón MVP es especialmente importante en ASP.NET WebForms, donde el modelo tradicional de code-behind acoplado dificulta enormemente el testing. MVP separa la lógica de presentación de la interfaz de usuario, delegando incluso los eventos y el ciclo de vida de la página al Presenter.

Este patrón permite:
- Testear la lógica de presentación sin necesidad de un servidor web
- Delegar completamente los eventos del ciclo de vida (Page_Load, Page_Init, etc.)
- Mantener el code-behind lo más delgado posible (Passive View)

### Ventajas para Testing

- Permite testear toda la lógica de presentación sin WebForms
- Elimina la dependencia del contexto HTTP en las pruebas
- Facilita el mocking de la vista completa
- Simplifica el testing de eventos y ciclo de vida de la página

### Componentes del Patrón

1. **Model**: Entidades y lógica de negocio
2. **View (Interfaz)**: Contrato que define qué puede hacer la vista
3. **View (Implementación)**: El code-behind de WebForms (Passive View)
4. **Presenter**: Toda la lógica de presentación y manejo de eventos

### Ejemplo Completo en .NET Framework

```csharp
// ===============================
// MODELO
// ===============================

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public bool IsActive { get; set; }
}

public class ProductSearchCriteria
{
    public string SearchTerm { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
}

// ===============================
// INTERFACES DE SERVICIOS
// ===============================

public interface IProductService
{
    IEnumerable<Product> SearchProducts(ProductSearchCriteria criteria);
    Product GetProductById(int id);
    void SaveProduct(Product product);
    void DeleteProduct(int id);
}

public interface IMessageService
{
    void ShowSuccess(string message);
    void ShowError(string message);
    void ShowWarning(string message);
}

// ===============================
// INTERFAZ DE LA VISTA
// ===============================

/// <summary>
/// Interfaz que define todas las operaciones que el Presenter
/// puede realizar sobre la vista. La vista es completamente pasiva.
/// </summary>
public interface IProductListView
{
    // Propiedades para binding de datos
    IEnumerable<Product> ProductList { set; }
    bool ShowLoadingIndicator { set; }
    bool ShowEmptyMessage { set; }
    string EmptyMessage { set; }
    
    // Propiedades de criterios de búsqueda
    string SearchTerm { get; set; }
    string MinPrice { get; set; }
    string MaxPrice { get; set; }
    
    // Propiedades de estado
    bool SearchButtonEnabled { set; }
    bool ClearButtonEnabled { set; }
    
    // Eventos del ciclo de vida delegados al Presenter
    event EventHandler ViewLoad;
    event EventHandler ViewInit;
    event EventHandler ViewPreRender;
    
    // Eventos de acciones del usuario
    event EventHandler SearchButtonClick;
    event EventHandler ClearButtonClick;
    event EventHandler<ProductEventArgs> EditProductClick;
    event EventHandler<ProductEventArgs> DeleteProductClick;
    event EventHandler<ProductEventArgs> ViewProductClick;
}

/// <summary>
/// EventArgs personalizado para eventos de productos
/// </summary>
public class ProductEventArgs : EventArgs
{
    public int ProductId { get; set; }
    
    public ProductEventArgs(int productId)
    {
        ProductId = productId;
    }
}

// ===============================
// PRESENTER
// ===============================

/// <summary>
/// El Presenter contiene TODA la lógica de presentación.
/// Maneja el ciclo de vida de la vista y todos los eventos.
/// Es completamente testeable sin dependencias de WebForms.
/// </summary>
public class ProductListPresenter
{
    private readonly IProductListView _view;
    private readonly IProductService _productService;
    private readonly IMessageService _messageService;

    public ProductListPresenter(
        IProductListView view,
        IProductService productService,
        IMessageService messageService)
    {
        _view = view;
        _productService = productService;
        _messageService = messageService;

        // Suscribirse a TODOS los eventos de la vista
        SubscribeToViewEvents();
    }

    /// <summary>
    /// Suscripción centralizada a todos los eventos de la vista
    /// </summary>
    private void SubscribeToViewEvents()
    {
        // Eventos del ciclo de vida
        _view.ViewInit += OnViewInit;
        _view.ViewLoad += OnViewLoad;
        _view.ViewPreRender += OnViewPreRender;
        
        // Eventos de usuario
        _view.SearchButtonClick += OnSearchButtonClick;
        _view.ClearButtonClick += OnClearButtonClick;
        _view.EditProductClick += OnEditProductClick;
        _view.DeleteProductClick += OnDeleteProductClick;
        _view.ViewProductClick += OnViewProductClick;
    }

    // ===============================
    // MANEJADORES DEL CICLO DE VIDA
    // ===============================

    /// <summary>
    /// Lógica para el evento Page_Init
    /// </summary>
    private void OnViewInit(object sender, EventArgs e)
    {
        // Inicialización temprana si es necesaria
        _view.ShowLoadingIndicator = false;
        _view.ShowEmptyMessage = false;
    }

    /// <summary>
    /// Lógica para el evento Page_Load
    /// Aquí se maneja el IsPostBack a través del Presenter
    /// </summary>
    private void OnViewLoad(object sender, EventArgs e)
    {
        // En MVP, verificamos si es la primera carga
        // Esta información puede venir de la vista
        if (!IsPostBack())
        {
            InitializeView();
        }
    }

    /// <summary>
    /// Lógica para el evento Page_PreRender
    /// </summary>
    private void OnViewPreRender(object sender, EventArgs e)
    {
        // Lógica justo antes del renderizado
        UpdateViewState();
    }

    /// <summary>
    /// Verificar si es PostBack (la vista proporciona esta info)
    /// </summary>
    private bool IsPostBack()
    {
        // En la implementación real, la vista WebForms
        // expondría esta información a través de una propiedad
        // Por ahora, asumimos carga inicial
        return false;
    }

    /// <summary>
    /// Inicialización de la vista en la primera carga
    /// </summary>
    private void InitializeView()
    {
        _view.SearchTerm = string.Empty;
        _view.MinPrice = string.Empty;
        _view.MaxPrice = string.Empty;
        
        LoadAllProducts();
    }

    /// <summary>
    /// Actualizar el estado de controles antes del renderizado
    /// </summary>
    private void UpdateViewState()
    {
        bool hasSearchCriteria = 
            !string.IsNullOrWhiteSpace(_view.SearchTerm) ||
            !string.IsNullOrWhiteSpace(_view.MinPrice) ||
            !string.IsNullOrWhiteSpace(_view.MaxPrice);

        _view.ClearButtonEnabled = hasSearchCriteria;
        _view.SearchButtonEnabled = true;
    }

    // ===============================
    // MANEJADORES DE EVENTOS DE USUARIO
    // ===============================

    private void OnSearchButtonClick(object sender, EventArgs e)
    {
        try
        {
            _view.ShowLoadingIndicator = true;
            
            var criteria = BuildSearchCriteria();
            var products = _productService.SearchProducts(criteria);
            
            DisplayProducts(products);
        }
        catch (Exception ex)
        {
            _messageService.ShowError($"Error al buscar productos: {ex.Message}");
        }
        finally
        {
            _view.ShowLoadingIndicator = false;
        }
    }

    private void OnClearButtonClick(object sender, EventArgs e)
    {
        _view.SearchTerm = string.Empty;
        _view.MinPrice = string.Empty;
        _view.MaxPrice = string.Empty;
        
        LoadAllProducts();
    }

    private void OnEditProductClick(object sender, ProductEventArgs e)
    {
        // Navegar a página de edición o mostrar modal
        // En un escenario real, podrías tener un INavigationService
        _messageService.ShowSuccess($"Editando producto ID: {e.ProductId}");
    }

    private void OnDeleteProductClick(object sender, ProductEventArgs e)
    {
        try
        {
            _productService.DeleteProduct(e.ProductId);
            _messageService.ShowSuccess("Producto eliminado correctamente");
            
            // Recargar la lista
            LoadAllProducts();
        }
        catch (Exception ex)
        {
            _messageService.ShowError($"Error al eliminar producto: {ex.Message}");
        }
    }

    private void OnViewProductClick(object sender, ProductEventArgs e)
    {
        var product = _productService.GetProductById(e.ProductId);
        
        if (product != null)
        {
            _messageService.ShowSuccess(
                $"Producto: {product.Name} - Precio: {product.Price:C}");
        }
    }

    // ===============================
    // MÉTODOS AUXILIARES
    // ===============================

    private void LoadAllProducts()
    {
        try
        {
            _view.ShowLoadingIndicator = true;
            
            var allProducts = _productService.SearchProducts(
                new ProductSearchCriteria());
            
            DisplayProducts(allProducts);
        }
        catch (Exception ex)
        {
            _messageService.ShowError($"Error al cargar productos: {ex.Message}");
        }
        finally
        {
            _view.ShowLoadingIndicator = false;
        }
    }

    private ProductSearchCriteria BuildSearchCriteria()
    {
        var criteria = new ProductSearchCriteria
        {
            SearchTerm = _view.SearchTerm
        };

        // Parsear precios si están presentes
        if (!string.IsNullOrWhiteSpace(_view.MinPrice))
        {
            if (decimal.TryParse(_view.MinPrice, out decimal minPrice))
            {
                criteria.MinPrice = minPrice;
            }
        }

        if (!string.IsNullOrWhiteSpace(_view.MaxPrice))
        {
            if (decimal.TryParse(_view.MaxPrice, out decimal maxPrice))
            {
                criteria.MaxPrice = maxPrice;
            }
        }

        return criteria;
    }

    private void DisplayProducts(IEnumerable<Product> products)
    {
        var productList = products?.ToList() ?? new List<Product>();

        if (productList.Any())
        {
            _view.ProductList = productList;
            _view.ShowEmptyMessage = false;
        }
        else
        {
            _view.ProductList = new List<Product>();
            _view.ShowEmptyMessage = true;
            _view.EmptyMessage = "No se encontraron productos con los criterios especificados.";
        }
    }
}

// ===============================
// IMPLEMENTACIÓN DE LA VISTA (WEBFORMS)
// ===============================

/// <summary>
/// Code-behind de la página WebForms.
/// SOLO se encarga de:
/// 1. Implementar la interfaz IProductListView
/// 2. Delegar todos los eventos al Presenter
/// 3. Exponer propiedades para binding
/// 
/// NO contiene lógica de negocio ni de presentación.
/// </summary>
public partial class ProductList : System.Web.UI.Page, IProductListView
{
    private ProductListPresenter _presenter;

    // ===============================
    // IMPLEMENTACIÓN DE PROPIEDADES
    // ===============================

    public IEnumerable<Product> ProductList
    {
        set
        {
            gvProducts.DataSource = value;
            gvProducts.DataBind();
        }
    }

    public bool ShowLoadingIndicator
    {
        set { pnlLoading.Visible = value; }
    }

    public bool ShowEmptyMessage
    {
        set { pnlEmptyMessage.Visible = value; }
    }

    public string EmptyMessage
    {
        get { return lblEmptyMessage.Text; }
        set { lblEmptyMessage.Text = value; }
    }

    public string SearchTerm
    {
        get { return txtSearch.Text; }
        set { txtSearch.Text = value; }
    }

    public string MinPrice
    {
        get { return txtMinPrice.Text; }
        set { txtMinPrice.Text = value; }
    }

    public string MaxPrice
    {
        get { return txtMaxPrice.Text; }
        set { txtMaxPrice.Text = value; }
    }

    public bool SearchButtonEnabled
    {
        set { btnSearch.Enabled = value; }
    }

    public bool ClearButtonEnabled
    {
        set { btnClear.Enabled = value; }
    }

    // ===============================
    // EVENTOS DELEGADOS AL PRESENTER
    // ===============================

    public event EventHandler ViewLoad;
    public event EventHandler ViewInit;
    public event EventHandler ViewPreRender;
    public event EventHandler SearchButtonClick;
    public event EventHandler ClearButtonClick;
    public event EventHandler<ProductEventArgs> EditProductClick;
    public event EventHandler<ProductEventArgs> DeleteProductClick;
    public event EventHandler<ProductEventArgs> ViewProductClick;

    // ===============================
    // CICLO DE VIDA DE WEBFORMS
    // ===============================

    protected void Page_Init(object sender, EventArgs e)
    {
        // Crear el Presenter con inyección de dependencias
        // En producción, usar un contenedor IoC
        _presenter = CreatePresenter();
        
        // Delegar el evento al Presenter
        ViewInit?.Invoke(this, e);
    }

    protected void Page_Load(object sender, EventArgs e)
    {
        // Delegar el evento al Presenter
        ViewLoad?.Invoke(this, e);
    }

    protected void Page_PreRender(object sender, EventArgs e)
    {
        // Delegar el evento al Presenter
        ViewPreRender?.Invoke(this, e);
    }

    // ===============================
    // EVENTOS DE CONTROLES
    // ===============================

    protected void btnSearch_Click(object sender, EventArgs e)
    {
        // Delegar al Presenter
        SearchButtonClick?.Invoke(this, e);
    }

    protected void btnClear_Click(object sender, EventArgs e)
    {
        // Delegar al Presenter
        ClearButtonClick?.Invoke(this, e);
    }

    protected void gvProducts_RowCommand(object sender, GridViewCommandEventArgs e)
    {
        int productId = Convert.ToInt32(e.CommandArgument);
        var args = new ProductEventArgs(productId);

        switch (e.CommandName)
        {
            case "EditProduct":
                EditProductClick?.Invoke(this, args);
                break;
            case "DeleteProduct":
                DeleteProductClick?.Invoke(this, args);
                break;
            case "ViewProduct":
                ViewProductClick?.Invoke(this, args);
                break;
        }
    }

    // ===============================
    // FACTORY PARA EL PRESENTER
    // ===============================

    /// <summary>
    /// Crea el Presenter con todas sus dependencias.
    /// En producción, usar un contenedor IoC como Unity.
    /// </summary>
    private ProductListPresenter CreatePresenter()
    {
        // Obtener servicios del contenedor IoC o crear manualmente
        var productService = DependencyResolver.Current.GetService<IProductService>();
        var messageService = new WebFormsMessageService(this);

        return new ProductListPresenter(this, productService, messageService);
    }
}

// ===============================
// IMPLEMENTACIÓN DEL MESSAGE SERVICE PARA WEBFORMS
// ===============================

/// <summary>
/// Implementación del servicio de mensajes para WebForms
/// que usa el sistema de mensajes de ASP.NET
/// </summary>
public class WebFormsMessageService : IMessageService
{
    private readonly Page _page;

    public WebFormsMessageService(Page page)
    {
        _page = page;
    }

    public void ShowSuccess(string message)
    {
        ShowMessage(message, "success");
    }

    public void ShowError(string message)
    {
        ShowMessage(message, "error");
    }

    public void ShowWarning(string message)
    {
        ShowMessage(message, "warning");
    }

    private void ShowMessage(string message, string type)
    {
        var script = $@"
            <script type='text/javascript'>
                alert('{message.Replace("'", "\\'")}');
            </script>";
        
        _page.ClientScript.RegisterStartupScript(
            _page.GetType(),
            "MessageScript",
            script);
    }
}
```

### Marcado ASPX (ProductList.aspx)

```aspx
<%@ Page Language="C#" AutoEventWireup="true" 
    CodeBehind="ProductList.aspx.cs" Inherits="MyApp.ProductList" %>

<!DOCTYPE html>
<html>
<head runat="server">
    <title>Lista de Productos</title>
</head>
<body>
    <form id="form1" runat="server">
        <div>
            <h2>Gestión de Productos</h2>
            
            <!-- Panel de búsqueda -->
            <div class="search-panel">
                <asp:TextBox ID="txtSearch" runat="server" 
                    placeholder="Buscar productos..." />
                
                <asp:TextBox ID="txtMinPrice" runat="server" 
                    placeholder="Precio mínimo" />
                
                <asp:TextBox ID="txtMaxPrice" runat="server" 
                    placeholder="Precio máximo" />
                
                <asp:Button ID="btnSearch" runat="server" 
                    Text="Buscar" OnClick="btnSearch_Click" />
                
                <asp:Button ID="btnClear" runat="server" 
                    Text="Limpiar" OnClick="btnClear_Click" />
            </div>

            <!-- Panel de carga -->
            <asp:Panel ID="pnlLoading" runat="server" Visible="false">
                <p>Cargando productos...</p>
            </asp:Panel>

            <!-- Panel de mensaje vacío -->
            <asp:Panel ID="pnlEmptyMessage" runat="server" Visible="false">
                <asp:Label ID="lblEmptyMessage" runat="server" />
            </asp:Panel>

            <!-- GridView de productos -->
            <asp:GridView ID="gvProducts" runat="server" 
                AutoGenerateColumns="False"
                OnRowCommand="gvProducts_RowCommand">
                <Columns>
                    <asp:BoundField DataField="Id" HeaderText="ID" />
                    <asp:BoundField DataField="Name" HeaderText="Nombre" />
                    <asp:BoundField DataField="Price" HeaderText="Precio" 
                        DataFormatString="{0:C}" />
                    <asp:BoundField DataField="Stock" HeaderText="Stock" />
                    
                    <asp:TemplateField HeaderText="Acciones">
                        <ItemTemplate>
                            <asp:Button runat="server" 
                                Text="Ver" 
                                CommandName="ViewProduct"
                                CommandArgument='<%# Eval("Id") %>' />
                            
                            <asp:Button runat="server" 
                                Text="Editar" 
                                CommandName="EditProduct"
                                CommandArgument='<%# Eval("Id") %>' />
                            
                            <asp:Button runat="server" 
                                Text="Eliminar" 
                                CommandName="DeleteProduct"
                                CommandArgument='<%# Eval("Id") %>'
                                OnClientClick="return confirm('¿Eliminar?');" />
                        </ItemTemplate>
                    </asp:TemplateField>
                </Columns>
            </asp:GridView>
        </div>
    </form>
</body>
</html>
```

### Pruebas Unitarias Completas

```csharp
[TestClass]
public class ProductListPresenterTests
{
    private Mock<IProductListView> _mockView;
    private Mock<IProductService> _mockProductService;
    private Mock<IMessageService> _mockMessageService;
    private ProductListPresenter _presenter;

    [TestInitialize]
    public void Setup()
    {
        _mockView = new Mock<IProductListView>();
        _mockProductService = new Mock<IProductService>();
        _mockMessageService = new Mock<IMessageService>();

        // Configurar propiedades de la vista
        _mockView.SetupProperty(v => v.SearchTerm);
        _mockView.SetupProperty(v => v.MinPrice);
        _mockView.SetupProperty(v => v.MaxPrice);
        _mockView.SetupProperty(v => v.ShowLoadingIndicator);
        _mockView.SetupProperty(v => v.ShowEmptyMessage);
        _mockView.SetupProperty(v => v.EmptyMessage);
        _mockView.SetupProperty(v => v.SearchButtonEnabled);
        _mockView.SetupProperty(v => v.ClearButtonEnabled);

        // Crear el presenter (esto suscribe automáticamente los eventos)
        _presenter = new ProductListPresenter(
            _mockView.Object,
            _mockProductService.Object,
            _mockMessageService.Object);
    }

    // ===============================
    // TESTS DEL CICLO DE VIDA
    // ===============================

    [TestMethod]
    public void ViewInit_InitializesViewCorrectly()
    {
        // Act - Disparar el evento ViewInit
        _mockView.Raise(v => v.ViewInit += null, EventArgs.Empty);

        // Assert
        Assert.IsFalse(_mockView.Object.ShowLoadingIndicator);
        Assert.IsFalse(_mockView.Object.ShowEmptyMessage);
    }

    [TestMethod]
    public void ViewLoad_FirstLoad_InitializesViewAndLoadsProducts()
    {
        // Arrange
        var products = new List<Product>
        {
            new Product { Id = 1, Name = "Product 1", Price = 10m },
            new Product { Id = 2, Name = "Product 2", Price = 20m }
        };

        _mockProductService
            .Setup(s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()))
            .Returns(products);

        // Act - Disparar el evento ViewLoad
        _mockView.Raise(v => v.ViewLoad += null, EventArgs.Empty);

        // Assert
        _mockView.VerifySet(v => v.SearchTerm = string.Empty);
        _mockView.VerifySet(v => v.MinPrice = string.Empty);
        _mockView.VerifySet(v => v.MaxPrice = string.Empty);
        _mockView.VerifySet(v => v.ProductList = It.IsAny<IEnumerable<Product>>());
        _mockProductService.Verify(
            s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()), 
            Times.Once);
    }

    [TestMethod]
    public void ViewPreRender_WithSearchCriteria_EnablesClearButton()
    {
        // Arrange
        _mockView.Object.SearchTerm = "test";

        // Act
        _mockView.Raise(v => v.ViewPreRender += null, EventArgs.Empty);

        // Assert
        _mockView.VerifySet(v => v.ClearButtonEnabled = true);
        _mockView.VerifySet(v => v.SearchButtonEnabled = true);
    }

    // ===============================
    // TESTS DE BÚSQUEDA
    // ===============================

    [TestMethod]
    public void SearchButtonClick_WithValidCriteria_DisplaysProducts()
    {
        // Arrange
        _mockView.Object.SearchTerm = "laptop";
        _mockView.Object.MinPrice = "100";
        _mockView.Object.MaxPrice = "500";

        var expectedProducts = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop HP", Price = 350m }
        };

        _mockProductService
            .Setup(s => s.SearchProducts(It.Is<ProductSearchCriteria>(
                c => c.SearchTerm == "laptop" && 
                     c.MinPrice == 100m && 
                     c.MaxPrice == 500m)))
            .Returns(expectedProducts);

        // Act
        _mockView.Raise(v => v.SearchButtonClick += null, EventArgs.Empty);

        // Assert
        _mockView.VerifySet(v => v.ShowLoadingIndicator = true, Times.Once);
        _mockView.VerifySet(v => v.ShowLoadingIndicator = false, Times.Once);
        _mockView.VerifySet(v => v.ProductList = expectedProducts);
        _mockView.VerifySet(v => v.ShowEmptyMessage = false);
    }

    [TestMethod]
    public void SearchButtonClick_NoResults_ShowsEmptyMessage()
    {
        // Arrange
        _mockView.Object.SearchTerm = "nonexistent";

        _mockProductService
            .Setup(s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()))
            .Returns(new List<Product>());

        // Act
        _mockView.Raise(v => v.SearchButtonClick += null, EventArgs.Empty);

        // Assert
        _mockView.VerifySet(v => v.ShowEmptyMessage = true);
        _mockView.VerifySet(
            v => v.EmptyMessage = 
                "No se encontraron productos con los criterios especificados.");
    }

    [TestMethod]
    public void SearchButtonClick_ServiceThrowsException_ShowsErrorMessage()
    {
        // Arrange
        _mockProductService
            .Setup(s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()))
            .Throws(new Exception("Database error"));

        // Act
        _mockView.Raise(v => v.SearchButtonClick += null, EventArgs.Empty);

        // Assert
        _mockMessageService.Verify(
            m => m.ShowError(It.Is<string>(s => s.Contains("Database error"))),
            Times.Once);
        _mockView.VerifySet(v => v.ShowLoadingIndicator = false);
    }

    // ===============================
    // TESTS DE LIMPIAR BÚSQUEDA
    // ===============================

    [TestMethod]
    public void ClearButtonClick_ClearsCriteriaAndReloadsProducts()
    {
        // Arrange
        _mockView.Object.SearchTerm = "test";
        _mockView.Object.MinPrice = "10";
        _mockView.Object.MaxPrice = "100";

        var allProducts = new List<Product>
        {
            new Product { Id = 1, Name = "Product 1" },
            new Product { Id = 2, Name = "Product 2" }
        };

        _mockProductService
            .Setup(s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()))
            .Returns(allProducts);

        // Act
        _mockView.Raise(v => v.ClearButtonClick += null, EventArgs.Empty);

        // Assert
        Assert.AreEqual(string.Empty, _mockView.Object.SearchTerm);
        Assert.AreEqual(string.Empty, _mockView.Object.MinPrice);
        Assert.AreEqual(string.Empty, _mockView.Object.MaxPrice);
        _mockView.VerifySet(v => v.ProductList = allProducts);
    }

    // ===============================
    // TESTS DE ACCIONES DE PRODUCTOS
    // ===============================

    [TestMethod]
    public void EditProductClick_RaisesCorrectMessage()
    {
        // Arrange
        var productId = 5;
        var args = new ProductEventArgs(productId);

        // Act
        _mockView.Raise(v => v.EditProductClick += null, _mockView.Object, args);

        // Assert
        _mockMessageService.Verify(
            m => m.ShowSuccess($"Editando producto ID: {productId}"),
            Times.Once);
    }

    [TestMethod]
    public void DeleteProductClick_DeletesAndReloadsProducts()
    {
        // Arrange
        var productId = 3;
        var args = new ProductEventArgs(productId);

        var remainingProducts = new List<Product>
        {
            new Product { Id = 1, Name = "Product 1" }
        };

        _mockProductService
            .Setup(s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()))
            .Returns(remainingProducts);

        // Act
        _mockView.Raise(
            v => v.DeleteProductClick += null, 
            _mockView.Object, 
            args);

        // Assert
        _mockProductService.Verify(s => s.DeleteProduct(productId), Times.Once);
        _mockMessageService.Verify(
            m => m.ShowSuccess("Producto eliminado correctamente"),
            Times.Once);
        _mockView.VerifySet(v => v.ProductList = remainingProducts);
    }

    [TestMethod]
    public void DeleteProductClick_ServiceFails_ShowsError()
    {
        // Arrange
        var productId = 3;
        var args = new ProductEventArgs(productId);

        _mockProductService
            .Setup(s => s.DeleteProduct(productId))
            .Throws(new Exception("Cannot delete"));

        // Act
        _mockView.Raise(
            v => v.DeleteProductClick += null,
            _mockView.Object,
            args);

        // Assert
        _mockMessageService.Verify(
            m => m.ShowError(It.Is<string>(s => s.Contains("Cannot delete"))),
            Times.Once);
    }

    [TestMethod]
    public void ViewProductClick_ProductExists_ShowsProductDetails()
    {
        // Arrange
        var productId = 7;
        var args = new ProductEventArgs(productId);
        var product = new Product 
        { 
            Id = productId, 
            Name = "Test Product", 
            Price = 99.99m 
        };

        _mockProductService
            .Setup(s => s.GetProductById(productId))
            .Returns(product);

        // Act
        _mockView.Raise(
            v => v.ViewProductClick += null,
            _mockView.Object,
            args);

        // Assert
        _mockProductService.Verify(s => s.GetProductById(productId), Times.Once);
        _mockMessageService.Verify(
            m => m.ShowSuccess(It.Is<string>(
                s => s.Contains("Test Product") && s.Contains("99"))),
            Times.Once);
    }

    // ===============================
    // TESTS DE VALIDACIÓN DE PRECIOS
    // ===============================

    [TestMethod]
    public void SearchButtonClick_InvalidPriceFormat_IgnoresInvalidPrice()
    {
        // Arrange
        _mockView.Object.MinPrice = "invalid";
        _mockView.Object.MaxPrice = "100";

        ProductSearchCriteria capturedCriteria = null;
        _mockProductService
            .Setup(s => s.SearchProducts(It.IsAny<ProductSearchCriteria>()))
            .Callback<ProductSearchCriteria>(c => capturedCriteria = c)
            .Returns(new List<Product>());

        // Act
        _mockView.Raise(v => v.SearchButtonClick += null, EventArgs.Empty);

        // Assert
        Assert.IsNull(capturedCriteria.MinPrice); // Precio inválido ignorado
        Assert.AreEqual(100m, capturedCriteria.MaxPrice);
    }
}
```

### Ventajas del Patrón MVP en WebForms

1. **Testabilidad completa**: Todo el código se puede testear sin IIS ni contexto HTTP
2. **Separación de responsabilidades**: La vista es completamente pasiva
3. **Delegación del ciclo de vida**: Page_Load, Page_Init, etc. se manejan en el Presenter
4. **Código reutilizable**: El Presenter puede usarse con diferentes tecnologías de UI
5. **Mantenibilidad**: La lógica está centralizada y es fácil de encontrar

## 9. Patrón Null Object

### Descripción

El patrón Null Object proporciona un objeto con comportamiento neutral o predeterminado en lugar de null. Elimina la necesidad de verificaciones constantes de null.

### Ventajas para Testing

- Elimina NullReferenceException en pruebas
- Simplifica la configuración de pruebas con valores por defecto
- Facilita el testing de flujos alternativos sin complicar el código

### Ejemplo en .NET Framework

```csharp
// Interfaz común
public interface ILogger
{
    void Log(string message);
    void LogError(string message);
    void LogWarning(string message);
}

// Implementación real
public class FileLogger : ILogger
{
    private readonly string _filePath;

    public FileLogger(string filePath)
    {
        _filePath = filePath;
    }

    public void Log(string message)
    {
        File.AppendAllText(_filePath, $"{DateTime.Now}: {message}\n");
    }

    public void LogError(string message)
    {
        File.AppendAllText(_filePath, $"{DateTime.Now} ERROR: {message}\n");
    }

    public void LogWarning(string message)
    {
        File.AppendAllText(_filePath, $"{DateTime.Now} WARNING: {message}\n");
    }
}

// Null Object - no hace nada pero cumple el contrato
public class NullLogger : ILogger
{
    public void Log(string message)
    {
        // Intencionalmente vacío
    }

    public void LogError(string message)
    {
        // Intencionalmente vacío
    }

    public void LogWarning(string message)
    {
        // Intencionalmente vacío
    }
}

// Servicio que usa el logger
public class UserService
{
    private readonly ILogger _logger;
    private readonly IUserRepository _userRepository;

    public UserService(IUserRepository userRepository, ILogger logger = null)
    {
        _userRepository = userRepository;
        // Usar NullLogger si no se proporciona uno
        _logger = logger ?? new NullLogger();
    }

    public User CreateUser(string username, string email)
    {
        _logger.Log($"Creando usuario: {username}");

        try
        {
            if (string.IsNullOrWhiteSpace(username))
            {
                _logger.LogError("El nombre de usuario no puede estar vacío");
                throw new ArgumentException("El nombre de usuario no puede estar vacío");
            }

            var user = new User
            {
                Username = username,
                Email = email,
                CreatedDate = DateTime.Now
            };

            _userRepository.Add(user);
            _logger.Log($"Usuario creado exitosamente: {username}");

            return user;
        }
        catch (Exception ex)
        {
            _logger.LogError($"Error al crear usuario: {ex.Message}");
            throw;
        }
    }

    public void DeleteUser(int userId)
    {
        _logger.Log($"Eliminando usuario con ID: {userId}");
        
        var user = _userRepository.GetById(userId);
        if (user == null)
        {
            _logger.LogWarning($"Usuario no encontrado: {userId}");
            return;
        }

        _userRepository.Delete(userId);
        _logger.Log($"Usuario eliminado: {userId}");
    }
}

// Modelos de apoyo
public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
    public DateTime CreatedDate { get; set; }
}

public interface IUserRepository
{
    void Add(User user);
    User GetById(int id);
    void Delete(int id);
}
```

### Pruebas Unitarias

```csharp
[TestClass]
public class UserServiceTests
{
    [TestMethod]
    public void CreateUser_WithNullLogger_WorksCorrectly()
    {
        // Arrange
        var mockRepository = new Mock<IUserRepository>();
        // No pasamos logger - usará NullLogger internamente
        var service = new UserService(mockRepository.Object);

        // Act
        var user = service.CreateUser("testuser", "test@example.com");

        // Assert
        Assert.IsNotNull(user);
        Assert.AreEqual("testuser", user.Username);
        mockRepository.Verify(r => r.Add(It.IsAny<User>()), Times.Once);
        // No hay NullReferenceException aunque no hay logger
    }

    [TestMethod]
    public void CreateUser_WithRealLogger_LogsMessages()
    {
        // Arrange
        var mockRepository = new Mock<IUserRepository>();
        var mockLogger = new Mock<ILogger>();
        
        var service = new UserService(mockRepository.Object, mockLogger.Object);

        // Act
        service.CreateUser("testuser", "test@example.com");

        // Assert
        mockLogger.Verify(l => l.Log(It.IsAny<string>()), Times.AtLeast(2));
        mockRepository.Verify(r => r.Add(It.IsAny<User>()), Times.Once);
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void CreateUser_EmptyUsername_ThrowsAndLogs()
    {
        // Arrange
        var mockRepository = new Mock<IUserRepository>();
        var mockLogger = new Mock<ILogger>();
        
        var service = new UserService(mockRepository.Object, mockLogger.Object);

        try
        {
            // Act
            service.CreateUser("", "test@example.com");
        }
        finally
        {
            // Assert
            mockLogger.Verify(l => l.LogError(It.IsAny<string>()), Times.Once);
            mockRepository.Verify(r => r.Add(It.IsAny<User>()), Times.Never);
        }
    }

    [TestMethod]
    public void DeleteUser_UserNotFound_LogsWarning()
    {
        // Arrange
        var mockRepository = new Mock<IUserRepository>();
        var mockLogger = new Mock<ILogger>();
        
        mockRepository.Setup(r => r.GetById(999)).Returns((User)null);
        
        var service = new UserService(mockRepository.Object, mockLogger.Object);

        // Act
        service.DeleteUser(999);

        // Assert
        mockLogger.Verify(l => l.LogWarning(It.IsAny<string>()), Times.Once);
        mockRepository.Verify(r => r.Delete(It.IsAny<int>()), Times.Never);
    }

    [TestMethod]
    public void NullLogger_DoesNotThrowExceptions()
    {
        // Arrange
        var nullLogger = new NullLogger();

        // Act & Assert - no debe lanzar excepciones
        nullLogger.Log("Test message");
        nullLogger.LogError("Error message");
        nullLogger.LogWarning("Warning message");
    }
}
```

---

## Mejores Prácticas

### 1. Combinar Patrones

Los patrones funcionan mejor cuando se combinan estratégicamente:

```csharp
// Repository + Factory + Strategy
public class OrderProcessingService
{
    private readonly IOrderRepository _repository;
    private readonly IShippingStrategyFactory _shippingFactory;
    private readonly IPaymentGateway _paymentGateway;

    public OrderProcessingService(
        IOrderRepository repository,
        IShippingStrategyFactory shippingFactory,
        IPaymentGateway paymentGateway)
    {
        _repository = repository;
        _shippingFactory = shippingFactory;
        _paymentGateway = paymentGateway;
    }

    public ProcessResult ProcessOrder(OrderRequest request)
    {
        // Obtener estrategia de envío
        var shippingStrategy = _shippingFactory.CreateStrategy(request.ShippingType);
        
        // Procesar pago
        var paymentResult = _paymentGateway.ProcessPayment(
            request.Total,
            request.PaymentInfo);

        if (!paymentResult.Success)
        {
            return ProcessResult.Failed("Error en el pago");
        }

        // Crear orden
        var order = new Order
        {
            CustomerId = request.CustomerId,
            Total = request.Total,
            ShippingCost = shippingStrategy.CalculateShippingCost(
                request.Total,
                request.Destination)
        };

        _repository.Add(order);

        return ProcessResult.Success(order.Id);
    }
}
```

### 2. Mantener Interfaces Simples

Las interfaces demasiado grandes dificultan el testing:

```csharp
// ❌ Mal - interfaz muy grande
public interface IUserService
{
    User CreateUser(string username);
    void DeleteUser(int id);
    void UpdateUser(User user);
    bool ValidateEmail(string email);
    void SendWelcomeEmail(User user);
    void LogUserActivity(int userId, string activity);
}

// ✅ Bien - interfaces segregadas (ISP)
public interface IUserRepository
{
    void Add(User user);
    void Update(User user);
    void Delete(int id);
    User GetById(int id);
}

public interface IEmailService
{
    void SendWelcomeEmail(User user);
}

public interface IUserValidator
{
    bool ValidateEmail(string email);
}
```

### 3. Usar Inyección de Dependencias

La inyección de dependencias es fundamental para todos estos patrones:

```csharp
// Configuración en aplicación .NET Framework con Unity Container
public class DependencyConfig
{
    public static void RegisterDependencies()
    {
        var container = new UnityContainer();

        // Registrar dependencias
        container.RegisterType<IUserRepository, UserRepository>();
        container.RegisterType<IEmailService, EmailService>();
        container.RegisterType<ILogger, FileLogger>();
        
        // Registrar con lifetime
        container.RegisterType<IDataCache, MemoryCache>(
            new ContainerControlledLifetimeManager());

        // Factories
        container.RegisterType<IShippingStrategyFactory, ShippingStrategyFactory>();

        DependencyResolver.SetResolver(new UnityDependencyResolver(container));
    }
}
```

### 4. Documentar Patrones Utilizados

```csharp
/// <summary>
/// Servicio de procesamiento de pedidos.
/// Implementa el patrón Repository para acceso a datos
/// y el patrón Strategy para cálculo de envíos.
/// </summary>
/// <remarks>
/// Este servicio es completamente testeable mediante inyección
/// de dependencias. Todos los colaboradores pueden ser reemplazados
/// con mocks en las pruebas unitarias.
/// </remarks>
public class OrderProcessingService
{
    // ...
}
```

---

## Conclusiones

Los patrones de diseño presentados en este documento son herramientas esenciales para crear código testeable en .NET Framework:

1. **Repository**: Aísla el acceso a datos y facilita el mocking de persistencia
2. **Strategy**: Permite intercambiar algoritmos y probar cada uno independientemente
3. **Factory**: Controla la creación de objetos complejos y facilita su testing
4. **Adapter**: Envuelve dependencias externas para hacerlas testeables
5. **Observer**: Permite probar notificaciones y eventos de forma aislada
6. **Decorator**: Añade funcionalidad testeable sin modificar código existente
7. **Template Method**: Define algoritmos con pasos testeables individualmente
8. **Null Object**: Elimina verificaciones de null y simplifica pruebas

### Beneficios Principales

- **Bajo acoplamiento**: Las clases dependen de abstracciones, no de implementaciones concretas
- **Alta cohesión**: Cada clase tiene una responsabilidad única y bien definida
- **Testabilidad**: Todas las dependencias pueden ser reemplazadas con mocks
- **Mantenibilidad**: El código es más fácil de entender y modificar
- **Escalabilidad**: Nuevas funcionalidades se añaden sin modificar código existente

### Recomendaciones Finales

- Aplique estos patrones de forma pragmática, no todos son necesarios en todos los casos
- Priorice la simplicidad sobre la perfección arquitectónica
- Use estos patrones en conjunto con principios SOLID
- Documente claramente qué patrones está utilizando y por qué
- Refactorice hacia patrones cuando el código lo necesite, no prematuramente

El dominio de estos patrones, combinado con frameworks de testing como MSTest, xUnit o NUnit, y herramientas de mocking como Moq o FakeItEasy, le permitirá crear aplicaciones .NET Framework robustas, mantenibles y completamente testeadas.

# Contenedores IoC y Ciclo de Vida de Dependencias

## Índice
1. [Introducción](#introducción)
2. [¿Qué es un Contenedor IoC?](#qué-es-un-contenedor-ioc)
3. [Contenedores IoC en .NET](#contenedores-ioc-en-net)
4. [Ciclos de Vida de Dependencias](#ciclos-de-vida-de-dependencias)
5. [Implementación Práctica](#implementación-práctica)
6. [Mejores Prácticas](#mejores-prácticas)
7. [Errores Comunes](#errores-comunes)
8. [Ejercicios Prácticos](#ejercicios-prácticos)

---

## Introducción

Los contenedores de Inversión de Control (IoC) son componentes fundamentales en el desarrollo moderno de aplicaciones .NET. Permiten gestionar automáticamente la creación, configuración y ciclo de vida de las dependencias, facilitando la implementación del principio de Inversión de Dependencias (el "D" de SOLID).

### Objetivos de Aprendizaje
- Comprender qué es un contenedor IoC y cómo funciona
- Conocer los principales contenedores disponibles en .NET
- Dominar los diferentes ciclos de vida de dependencias
- Aplicar buenas prácticas en la gestión de dependencias
- Identificar y evitar problemas comunes

---

## ¿Qué es un Contenedor IoC?

### Definición

Un **Contenedor IoC** (Inversion of Control Container) o **DI Container** (Dependency Injection Container) es un framework que automatiza el proceso de inyección de dependencias. Actúa como un "registro" donde configuramos qué implementaciones concretas deben usarse para cada abstracción, y se encarga de:

1. **Resolver** las dependencias cuando se solicitan
2. **Crear** instancias de las clases necesarias
3. **Gestionar** el ciclo de vida de los objetos
4. **Liberar** recursos cuando ya no son necesarios

### ¿Por qué usar un Contenedor IoC?

#### Ventajas

**1. Desacoplamiento**
```csharp
// Sin IoC - Acoplamiento directo
public class OrderService
{
    private readonly SqlOrderRepository _repository;
    
    public OrderService()
    {
        _repository = new SqlOrderRepository(); // Acoplamiento fuerte
    }
}

// Con IoC - Desacoplamiento
public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public OrderService(IOrderRepository repository)
    {
        _repository = repository; // Inyectado por el contenedor
    }
}
```

**2. Facilita el Testing**
```csharp
// En producción: el contenedor inyecta SqlOrderRepository
// En pruebas: inyectamos un mock
var mockRepository = new Mock<IOrderRepository>();
var service = new OrderService(mockRepository.Object);
```

**3. Gestión Centralizada**
```csharp
// Toda la configuración en un solo lugar
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddTransient<IEmailService, SmtpEmailService>();
services.AddSingleton<IConfiguration, AppConfiguration>();
```

**4. Gestión Automática del Ciclo de Vida**
El contenedor se encarga de crear y destruir objetos según su configuración.

---

## Contenedores IoC en .NET

### 1. Microsoft.Extensions.DependencyInjection (Built-in)

El contenedor nativo de .NET, incluido en ASP.NET Core y disponible para cualquier aplicación .NET.

#### Características
- Ligero y rápido
- Integrado en el framework
- Suficiente para la mayoría de escenarios
- Limitado en características avanzadas

#### Configuración Básica

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var builder = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        // Registro de servicios
        services.AddTransient<IEmailService, EmailService>();
        services.AddScoped<IOrderRepository, OrderRepository>();
        services.AddSingleton<IConfiguration, AppConfiguration>();
    });

var host = builder.Build();
```

#### En aplicaciones .NET Framework

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        var services = new ServiceCollection();
        
        // Registrar servicios
        services.AddScoped<IProductService, ProductService>();
        services.AddTransient<INotificationService, NotificationService>();
        services.AddSingleton<ICacheService, MemoryCacheService>();
        
        var serviceProvider = services.BuildServiceProvider();
    }
}
```

### 2. Autofac

Uno de los contenedores más populares y con más características.

#### Instalación
```bash
dotnet add package Autofac
dotnet add package Autofac.Extensions.DependencyInjection
```

#### Características
- Registro por convención
- Módulos para organizar configuraciones
- Inyección de propiedades
- Decoradores
- Interceptores

#### Ejemplo de Uso en .NET Framework

```csharp
using Autofac;

public class AutofacConfig
{
    public static IContainer ConfigureContainer()
    {
        var builder = new ContainerBuilder();
        
        // Registro individual
        builder.RegisterType<EmailService>()
            .As<IEmailService>()
            .InstancePerLifetimeScope();

        // Registro por convención
        builder.RegisterAssemblyTypes(typeof(AutofacConfig).Assembly)
            .Where(t => t.Name.EndsWith("Repository"))
            .AsImplementedInterfaces()
            .InstancePerLifetimeScope();

        // Registro de módulos
        builder.RegisterModule<DataAccessModule>();
        
        return builder.Build();
    }
}
```

#### Módulos de Autofac

```csharp
public class DataAccessModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<SqlContext>()
            .AsSelf()
            .InstancePerLifetimeScope();

        builder.RegisterAssemblyTypes(Assembly.GetExecutingAssembly())
            .Where(t => t.Name.EndsWith("Repository"))
            .AsImplementedInterfaces()
            .InstancePerLifetimeScope();
    }
}
```

### 3. Ninject

Contenedor IoC con sintaxis fluida y fácil de usar.

#### Instalación
```bash
dotnet add package Ninject
```

#### Ejemplo

```csharp
using Ninject;

var kernel = new StandardKernel();

// Registro de bindings
kernel.Bind<IEmailService>().To<EmailService>().InTransientScope();
kernel.Bind<IOrderRepository>().To<OrderRepository>().InRequestScope();
kernel.Bind<IConfiguration>().To<AppConfiguration>().InSingletonScope();

// Resolución
var emailService = kernel.Get<IEmailService>();
```

### 4. Simple Injector

Contenedor enfocado en rendimiento y diagnóstico.

#### Instalación
```bash
dotnet add package SimpleInjector
```

#### Ejemplo

```csharp
using SimpleInjector;

var container = new Container();

// Registro
container.Register<IEmailService, EmailService>(Lifestyle.Transient);
container.Register<IOrderRepository, OrderRepository>(Lifestyle.Scoped);
container.Register<IConfiguration, AppConfiguration>(Lifestyle.Singleton);

// Verificación
container.Verify();

// Resolución
var emailService = container.GetInstance<IEmailService>();
```

### 5. Unity

Contenedor de Microsoft, actualmente mantenido por la comunidad.

#### Instalación
```bash
dotnet add package Unity
```

#### Ejemplo

```csharp
using Unity;

var container = new UnityContainer();

// Registro
container.RegisterType<IEmailService, EmailService>(TypeLifetime.Transient);
container.RegisterType<IOrderRepository, OrderRepository>(TypeLifetime.Scoped);
container.RegisterType<IConfiguration, AppConfiguration>(TypeLifetime.Singleton);

// Resolución
var emailService = container.Resolve<IEmailService>();
```

### Comparación de Contenedores

| Característica | MS DI | Autofac | Ninject | Simple Injector | Unity |
|----------------|-------|---------|---------|-----------------|-------|
| Rendimiento | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Características | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Facilidad de uso | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Documentación | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Comunidad | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

---

## Ciclos de Vida de Dependencias

El **ciclo de vida** (lifetime) determina cuándo se crea y se destruye una instancia de un servicio. Elegir el ciclo de vida adecuado es crucial para el rendimiento, el uso de memoria y la correctitud de la aplicación.

### 1. Transient (Transitorio)

**Definición**: Se crea una nueva instancia cada vez que se solicita el servicio.

#### Características
- Instancia diferente en cada resolución
- Ciclo de vida más corto
- Mayor uso de memoria y CPU
- Ideal para servicios sin estado

#### Cuándo Usar
- Servicios ligeros sin estado
- Servicios que no mantienen datos entre llamadas
- Operaciones que deben ser independientes

#### Ejemplo

```csharp
// Registro
services.AddTransient<IEmailService, EmailService>();

// Uso
public class OrderController : Controller
{
    private readonly IEmailService _emailService1;
    private readonly IEmailService _emailService2;
    
    public OrderController(
        IEmailService emailService1,
        IEmailService emailService2)
    {
        _emailService1 = emailService1;
        _emailService2 = emailService2;
        
        // emailService1 y emailService2 son instancias DIFERENTES
        Console.WriteLine(ReferenceEquals(emailService1, emailService2)); // False
    }
}
```

#### Caso de Uso Real

```csharp
public interface IEmailService
{
    void SendEmail(string to, string subject, string body);
}

public class SmtpEmailService : IEmailService
{
    private readonly ILogService _logger;
    
    public SmtpEmailService(ILogService logger)
    {
        _logger = logger;
    }
    
    public void SendEmail(string to, string subject, string body)
    {
        _logger.LogInfo($"Enviando email a {to}");
        // Lógica de envío con SmtpClient
        using (var client = new SmtpClient())
        {
            // Configuración y envío
        }
    }
}

// Registro
services.AddTransient<IEmailService, SmtpEmailService>();
```

### 2. Scoped (Con Alcance)

**Definición**: Se crea una instancia por cada ámbito (scope). En aplicaciones web, un scope = una petición HTTP.

#### Características
- Una instancia por scope/petición
- Se destruye al finalizar el scope
- Ideal para operaciones con estado durante la petición
- Balance entre rendimiento y aislamiento

#### Cuándo Usar
- Repositorios y contextos de base de datos (DbContext)
- Servicios que mantienen estado durante una petición
- Servicios que necesitan ser consistentes dentro de una operación

#### Ejemplo

```csharp
// Registro
services.AddScoped<IOrderRepository, OrderRepository>();

// Uso en MVC
public class OrderController : Controller
{
    private readonly IOrderRepository _repository1;
    private readonly IOrderRepository _repository2;
    
    public OrderController(
        IOrderRepository repository1,
        IOrderRepository repository2)
    {
        _repository1 = repository1;
        _repository2 = repository2;
        
        // Dentro de la MISMA petición, son la MISMA instancia
        Console.WriteLine(ReferenceEquals(repository1, repository2)); // True
    }
    
    public ActionResult Create(Order order)
    {
        // Ambos repositorios comparten la misma instancia
        _repository1.Add(order);
        var count = _repository2.Count(); // Puede ver el cambio anterior
        
        return View();
    }
}
```

#### Caso de Uso Real con Entity Framework

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly MyDbContext _context;
    
    public OrderRepository(MyDbContext context)
    {
        _context = context;
    }
    
    public void Add(Order order)
    {
        _context.Orders.Add(order);
    }
    
    public void SaveChanges()
    {
        _context.SaveChanges();
    }
}

// Registro
services.AddScoped<MyDbContext>();
services.AddScoped<IOrderRepository, OrderRepository>();

// En una petición, todos los repositorios comparten el mismo DbContext
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ICustomerRepository _customerRepository;
    
    public OrderService(
        IOrderRepository orderRepository,
        ICustomerRepository customerRepository)
    {
        _orderRepository = orderRepository;
        _customerRepository = customerRepository;
        // Ambos repositorios usan el MISMO DbContext en esta petición
    }
}
```

### 3. Singleton

**Definición**: Se crea UNA ÚNICA instancia para toda la vida de la aplicación.

#### Características
- Una sola instancia para toda la aplicación
- Creada en el primer uso (lazy) o al iniciar
- Permanece viva hasta que termina la aplicación
- Mejor rendimiento, menor uso de memoria
- DEBE ser thread-safe

#### Cuándo Usar
- Configuraciones que no cambian
- Cachés en memoria
- Servicios sin estado que son costosos de crear
- Conexiones compartidas (con cuidado)
- Servicios de logging

#### Ejemplo

```csharp
// Registro
services.AddSingleton<IConfiguration, AppConfiguration>();

// Uso
public class OrderController : Controller
{
    private readonly IConfiguration _config1;
    private readonly IConfiguration _config2;
    
    public OrderController(
        IConfiguration config1,
        IConfiguration config2)
    {
        _config1 = config1;
        _config2 = config2;
        
        // SIEMPRE la misma instancia, en TODAS las peticiones
        Console.WriteLine(ReferenceEquals(config1, config2)); // True
    }
}
```

#### Caso de Uso Real - Cache Service

```csharp
public interface ICacheService
{
    void Set<T>(string key, T value);
    T Get<T>(string key);
}

public class MemoryCacheService : ICacheService
{
    private readonly ConcurrentDictionary<string, object> _cache;
    
    public MemoryCacheService()
    {
        _cache = new ConcurrentDictionary<string, object>();
    }
    
    public void Set<T>(string key, T value)
    {
        _cache[key] = value;
    }
    
    public T Get<T>(string key)
    {
        return _cache.TryGetValue(key, out var value) 
            ? (T)value 
            : default(T);
    }
}

// Registro
services.AddSingleton<ICacheService, MemoryCacheService>();

// Todos los controladores/servicios comparten la MISMA caché
```

### 4. InstancePerRequest (Autofac)

En Autofac, específico para aplicaciones web, equivalente a Scoped.

```csharp
builder.RegisterType<OrderRepository>()
    .As<IOrderRepository>()
    .InstancePerRequest();
```

### 5. InstancePerLifetimeScope (Autofac)

Más flexible que InstancePerRequest, crea instancias por cualquier scope definido.

```csharp
builder.RegisterType<OrderRepository>()
    .As<IOrderRepository>()
    .InstancePerLifetimeScope();
```

### Comparación Visual de Ciclos de Vida

```
TRANSIENT:    [A1] [A2] [A3] [A4]    // Nueva instancia cada vez
              ↓    ↓    ↓    ↓
              Usa  Usa  Usa  Usa
              Destruye ↓ ↓ ↓

SCOPED:       [------ Request 1 ------]  [------ Request 2 ------]
              [B1 compartido por todos]  [B2 compartido por todos]
              ↓                           ↓
              Destruye al finalizar       Destruye al finalizar

SINGLETON:    [========= C1 (toda la vida de la app) =========]
              ↓
              Nunca se destruye hasta que la app termina
```

### Tabla Comparativa

| Característica | Transient | Scoped | Singleton |
|----------------|-----------|--------|-----------|
| Instancias | Nueva cada vez | Una por scope | Una para toda la app |
| Vida útil | Muy corta | Duración del scope | Vida de la aplicación |
| Thread-safety | No requerida | No requerida | **REQUERIDA** |
| Uso de memoria | Alto | Medio | Bajo |
| Rendimiento | Menor | Medio | Mayor |
| Estado | Sin estado | Con estado (scope) | Sin estado o thread-safe |

---

## Implementación Práctica

### Estructura de un Proyecto con IoC

```
MiProyecto/
│
├── MiProyecto.Domain/
│   ├── Entities/
│   ├── Interfaces/
│   │   ├── IOrderRepository.cs
│   │   └── IProductRepository.cs
│   └── Services/
│       └── IOrderService.cs
│
├── MiProyecto.Infrastructure/
│   ├── Data/
│   │   └── MyDbContext.cs
│   ├── Repositories/
│   │   ├── OrderRepository.cs
│   │   └── ProductRepository.cs
│   └── DependencyInjection/
│       └── InfrastructureModule.cs
│
├── MiProyecto.Application/
│   ├── Services/
│   │   └── OrderService.cs
│   └── DependencyInjection/
│       └── ApplicationModule.cs
│
└── MiProyecto.Web/
    ├── Controllers/
    ├── Global.asax.cs
    └── App_Start/
        └── DependencyConfig.cs
```

### Ejemplo Completo con Unity Container

#### 1. Definir Interfaces (Domain Layer)

```csharp
namespace MiProyecto.Domain.Interfaces
{
    public interface IOrderRepository
    {
        Order GetById(int id);
        IEnumerable<Order> GetAll();
        void Add(Order order);
        void Update(Order order);
        void Delete(int id);
    }
    
    public interface IOrderService
    {
        OrderDto GetOrder(int id);
        void CreateOrder(CreateOrderDto dto);
    }
}
```

#### 2. Implementar Repositorios (Infrastructure Layer)

```csharp
namespace MiProyecto.Infrastructure.Repositories
{
    public class OrderRepository : IOrderRepository
    {
        private readonly MyDbContext _context;
        
        public OrderRepository(MyDbContext context)
        {
            _context = context;
        }
        
        public Order GetById(int id)
        {
            return _context.Orders.Find(id);
        }
        
        public IEnumerable<Order> GetAll()
        {
            return _context.Orders.ToList();
        }
        
        public void Add(Order order)
        {
            _context.Orders.Add(order);
            _context.SaveChanges();
        }
        
        public void Update(Order order)
        {
            _context.Entry(order).State = EntityState.Modified;
            _context.SaveChanges();
        }
        
        public void Delete(int id)
        {
            var order = _context.Orders.Find(id);
            if (order != null)
            {
                _context.Orders.Remove(order);
                _context.SaveChanges();
            }
        }
    }
}
```

#### 3. Implementar Servicios (Application Layer)

```csharp
namespace MiProyecto.Application.Services
{
    public class OrderService : IOrderService
    {
        private readonly IOrderRepository _repository;
        private readonly IMapper _mapper;
        
        public OrderService(IOrderRepository repository, IMapper mapper)
        {
            _repository = repository;
            _mapper = mapper;
        }
        
        public OrderDto GetOrder(int id)
        {
            var order = _repository.GetById(id);
            return _mapper.Map<OrderDto>(order);
        }
        
        public void CreateOrder(CreateOrderDto dto)
        {
            var order = _mapper.Map<Order>(dto);
            _repository.Add(order);
        }
    }
}
```

#### 4. Configurar el Contenedor (Web Layer)

```csharp
using Unity;
using Unity.Lifetime;
using Unity.AspNet.Mvc;
using System.Web.Mvc;

namespace MiProyecto.Web.App_Start
{
    public class DependencyConfig
    {
        public static void RegisterDependencies()
        {
            var container = new UnityContainer();
            
            // Infrastructure - Scoped (por petición HTTP)
            container.RegisterType<MyDbContext>(new HierarchicalLifetimeManager());
            container.RegisterType<IOrderRepository, OrderRepository>(
                new HierarchicalLifetimeManager());
            container.RegisterType<IProductRepository, ProductRepository>(
                new HierarchicalLifetimeManager());
            
            // Application Services - Scoped
            container.RegisterType<IOrderService, OrderService>(
                new HierarchicalLifetimeManager());
            
            // Singleton Services
            container.RegisterType<ICacheService, MemoryCacheService>(
                new ContainerControlledLifetimeManager());
            container.RegisterType<IConfiguration, AppConfiguration>(
                new ContainerControlledLifetimeManager());
            
            // Transient Services
            container.RegisterType<IEmailService, EmailService>(
                new TransientLifetimeManager());
            container.RegisterType<INotificationService, NotificationService>(
                new TransientLifetimeManager());
            
            // Configurar MVC para usar Unity
            DependencyResolver.SetResolver(new UnityDependencyResolver(container));
        }
    }
}
```

#### 5. Inicializar en Global.asax

```csharp
namespace MiProyecto.Web
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            
            // Configurar DI
            DependencyConfig.RegisterDependencies();
        }
    }
}
```

#### 6. Usar en Controladores

```csharp
namespace MiProyecto.Web.Controllers
{
    public class OrderController : Controller
    {
        private readonly IOrderService _orderService;
        private readonly ICacheService _cache;
        
        // Unity inyecta automáticamente las dependencias
        public OrderController(IOrderService orderService, ICacheService cache)
        {
            _orderService = orderService;
            _cache = cache;
        }
        
        public ActionResult Index()
        {
            var orders = _orderService.GetAllOrders();
            return View(orders);
        }
        
        public ActionResult Details(int id)
        {
            // Intentar obtener del caché primero
            var cacheKey = $"Order_{id}";
            var order = _cache.Get<OrderDto>(cacheKey);
            
            if (order == null)
            {
                order = _orderService.GetOrder(id);
                _cache.Set(cacheKey, order);
            }
            
            return View(order);
        }
        
        [HttpPost]
        public ActionResult Create(CreateOrderDto dto)
        {
            if (ModelState.IsValid)
            {
                _orderService.CreateOrder(dto);
                return RedirectToAction("Index");
            }
            
            return View(dto);
        }
    }
}
```

### Ejemplo con Autofac y Módulos

#### Módulo de Infraestructura

```csharp
using Autofac;

namespace MiProyecto.Infrastructure.DependencyInjection
{
    public class InfrastructureModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            // DbContext por petición
            builder.RegisterType<MyDbContext>()
                .AsSelf()
                .InstancePerRequest();
            
            // Registrar todos los repositorios automáticamente
            builder.RegisterAssemblyTypes(ThisAssembly)
                .Where(t => t.Name.EndsWith("Repository"))
                .AsImplementedInterfaces()
                .InstancePerRequest();
        }
    }
}
```

#### Módulo de Aplicación

```csharp
using Autofac;

namespace MiProyecto.Application.DependencyInjection
{
    public class ApplicationModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            // Registrar todos los servicios automáticamente
            builder.RegisterAssemblyTypes(ThisAssembly)
                .Where(t => t.Name.EndsWith("Service"))
                .AsImplementedInterfaces()
                .InstancePerRequest();
            
            // AutoMapper
            builder.Register(context => AutoMapperConfig.CreateMapper())
                .As<IMapper>()
                .SingleInstance();
        }
    }
}
```

#### Configuración en Global.asax con Autofac

```csharp
using Autofac;
using Autofac.Integration.Mvc;
using System.Web.Mvc;

namespace MiProyecto.Web
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            
            // Configurar Autofac
            var builder = new ContainerBuilder();
            
            // Registrar controladores
            builder.RegisterControllers(typeof(MvcApplication).Assembly);
            
            // Registrar módulos
            builder.RegisterModule<InfrastructureModule>();
            builder.RegisterModule<ApplicationModule>();
            
            // Servicios globales
            builder.RegisterType<MemoryCacheService>()
                .As<ICacheService>()
                .SingleInstance();
            
            builder.RegisterType<SmtpEmailService>()
                .As<IEmailService>()
                .InstancePerDependency();
            
            // Construir contenedor
            var container = builder.Build();
            
            // Configurar MVC
            DependencyResolver.SetResolver(new AutofacDependencyResolver(container));
        }
    }
}
```

---

## Mejores Prácticas

### 1. Principio de Registro Explícito

**✅ CORRECTO**
```csharp
// Registrar explícitamente cada servicio
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddScoped<IProductRepository, SqlProductRepository>();
services.AddTransient<IEmailService, SmtpEmailService>();
```

**❌ INCORRECTO**
```csharp
// Confiar en resolución automática sin configuración clara
var service = container.Resolve<OrderService>(); // Sin registro previo
```

### 2. Preferir Constructor Injection

**✅ CORRECTO**
```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    
    public OrderService(IOrderRepository repository, IEmailService emailService)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _emailService = emailService ?? throw new ArgumentNullException(nameof(emailService));
    }
}
```

**❌ INCORRECTO - Property Injection**
```csharp
public class OrderService
{
    // Puede ser null, no es claro que sea requerido
    public IOrderRepository Repository { get; set; }
    public IEmailService EmailService { get; set; }
}
```

### 3. No Abusar de Service Locator

**✅ CORRECTO**
```csharp
public class OrderController : Controller
{
    private readonly IOrderService _orderService;
    
    public OrderController(IOrderService orderService)
    {
        _orderService = orderService;
    }
}
```

**❌ INCORRECTO - Service Locator Anti-Pattern**
```csharp
public class OrderController : Controller
{
    public ActionResult Index()
    {
        // Anti-pattern: resolver dependencias manualmente
        var orderService = DependencyResolver.Current.GetService<IOrderService>();
        var orders = orderService.GetAll();
        return View(orders);
    }
}
```

### 4. Validar Dependencias Null

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public OrderService(IOrderRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
}
```

### 5. No Inyectar Contenedor

**❌ INCORRECTO**
```csharp
public class OrderService
{
    private readonly IUnityContainer _container;
    
    public OrderService(IUnityContainer container)
    {
        _container = container; // NO HACER ESTO
    }
    
    public void DoSomething()
    {
        var repo = _container.Resolve<IOrderRepository>();
    }
}
```

### 6. Usar el Ciclo de Vida Apropiado

```csharp
// DbContext: Scoped (por petición)
services.AddScoped<MyDbContext>();

// Repositories: Scoped (comparten DbContext)
services.AddScoped<IOrderRepository, OrderRepository>();

// Services sin estado: Transient
services.AddTransient<IEmailService, EmailService>();

// Caché: Singleton
services.AddSingleton<ICacheService, MemoryCacheService>();

// Configuración: Singleton
services.AddSingleton<IConfiguration, AppConfiguration>();
```

### 7. Resolver Dependencias Solo en la Raíz

```csharp
// ✅ CORRECTO - En Application_Start o punto de entrada
protected void Application_Start()
{
    DependencyConfig.RegisterDependencies();
}

// ❌ INCORRECTO - Resolver en medio del código
public void SomeMethod()
{
    var service = container.Resolve<IOrderService>();
}
```

### 8. Evitar Dependencias Circulares

**❌ INCORRECTO**
```csharp
public class ServiceA
{
    public ServiceA(ServiceB serviceB) { }
}

public class ServiceB
{
    public ServiceB(ServiceA serviceA) { } // Dependencia circular
}
```

**✅ CORRECTO - Refactorizar**
```csharp
public class ServiceA
{
    public ServiceA(ISharedDependency shared) { }
}

public class ServiceB
{
    public ServiceB(ISharedDependency shared) { }
}
```

### 9. Thread-Safety en Singletons

**✅ CORRECTO**
```csharp
public class MemoryCacheService : ICacheService
{
    // ConcurrentDictionary es thread-safe
    private readonly ConcurrentDictionary<string, object> _cache;
    
    public MemoryCacheService()
    {
        _cache = new ConcurrentDictionary<string, object>();
    }
    
    public void Set<T>(string key, T value)
    {
        _cache[key] = value; // Thread-safe
    }
}
```

**❌ INCORRECTO**
```csharp
public class CacheService : ICacheService
{
    // Dictionary NO es thread-safe
    private readonly Dictionary<string, object> _cache;
    
    public CacheService()
    {
        _cache = new Dictionary<string, object>();
    }
    
    public void Set<T>(string key, T value)
    {
        _cache[key] = value; // PELIGRO: No thread-safe
    }
}
```

### 10. Disposing Apropiado

```csharp
// El contenedor se encarga de Dispose automáticamente
public class OrderRepository : IOrderRepository, IDisposable
{
    private readonly MyDbContext _context;
    
    public OrderRepository(MyDbContext context)
    {
        _context = context;
    }
    
    public void Dispose()
    {
        // El contenedor llamará esto cuando finalice el scope
        _context?.Dispose();
    }
}
```

---

## Errores Comunes

### 1. Captive Dependency

**❌ ERROR: Inyectar Scoped/Transient en Singleton**

```csharp
// SINGLETON que depende de SCOPED - ¡ERROR!
public class MySingletonService
{
    private readonly MyDbContext _context; // DbContext es Scoped
    
    public MySingletonService(MyDbContext context)
    {
        _context = context; // El DbContext vivirá toda la vida de la app
    }
}

// Registro
services.AddSingleton<MySingletonService>(); // MALO
services.AddScoped<MyDbContext>(); // DbContext queda "capturado"
```

**Problema**: El DbContext vivirá toda la vida de la aplicación en lugar de por petición, causando problemas de concurrencia y memory leaks.

**✅ SOLUCIÓN 1: Inyectar Factory**

```csharp
public class MySingletonService
{
    private readonly Func<MyDbContext> _contextFactory;
    
    public MySingletonService(Func<MyDbContext> contextFactory)
    {
        _contextFactory = contextFactory;
    }
    
    public void DoSomething()
    {
        using (var context = _contextFactory())
        {
            // Usar el contexto
        }
    }
}
```

**✅ SOLUCIÓN 2: Cambiar a Scoped**

```csharp
// Si el servicio necesita DbContext, probablemente debería ser Scoped
services.AddScoped<MySingletonService>();
```

### 2. Resolver Dependencias en Constructor

**❌ INCORRECTO**

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public OrderService(IServiceProvider serviceProvider)
    {
        // NO resolver en el constructor
        _repository = serviceProvider.GetService<IOrderRepository>();
    }
}
```

**✅ CORRECTO**

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }
}
```

### 3. Olvidar Registrar Dependencias

**❌ ERROR**

```csharp
public class OrderController : Controller
{
    private readonly IOrderService _orderService;
    
    public OrderController(IOrderService orderService)
    {
        _orderService = orderService;
    }
}

// En DependencyConfig - ¡Olvidamos registrar IOrderService!
public static void RegisterDependencies()
{
    var container = new UnityContainer();
    container.RegisterType<IOrderRepository, OrderRepository>();
    // Falta: container.RegisterType<IOrderService, OrderService>();
}
```

**Resultado**: Exception al intentar crear el controlador.

### 4. Usar DbContext en Singleton

**❌ MUY PELIGROSO**

```csharp
// NUNCA hacer esto
services.AddSingleton<MyDbContext>();
```

**Problemas**:
- DbContext NO es thread-safe
- Causará errores de concurrencia
- Memory leaks
- Estado corrupto

**✅ CORRECTO**

```csharp
services.AddScoped<MyDbContext>(); // Uno por petición
```

### 5. Dependencias Circulares

**❌ ERROR**

```csharp
public class UserService
{
    public UserService(OrderService orderService) { }
}

public class OrderService
{
    public OrderService(UserService userService) { } // Circular
}
```

**Resultado**: Exception al intentar resolver.

**✅ SOLUCIÓN: Extraer dependencia común**

```csharp
public interface IDataAccess { }

public class UserService
{
    public UserService(IDataAccess dataAccess) { }
}

public class OrderService
{
    public OrderService(IDataAccess dataAccess) { }
}
```

### 6. No Disponer Recursos

**❌ INCORRECTO**

```csharp
public class FileService
{
    private FileStream _stream;
    
    public FileService()
    {
        _stream = File.OpenRead("file.txt");
        // Nunca se cierra
    }
}
```

**✅ CORRECTO**

```csharp
public class FileService : IDisposable
{
    private FileStream _stream;
    
    public FileService()
    {
        _stream = File.OpenRead("file.txt");
    }
    
    public void Dispose()
    {
        _stream?.Dispose();
    }
}
```

### 7. Múltiples Registros de la Misma Interfaz

```csharp
services.AddTransient<IEmailService, SmtpEmailService>();
services.AddTransient<IEmailService, SendGridEmailService>();

// ¿Cuál se inyectará? Depende del contenedor
// Unity: el último
// Autofac: el último
```

**✅ SOLUCIÓN: Usar nombres o tipos específicos**

```csharp
services.AddTransient<SmtpEmailService>();
services.AddTransient<SendGridEmailService>();

// O usar keyed services (Autofac)
builder.RegisterType<SmtpEmailService>()
    .Keyed<IEmailService>("smtp");
builder.RegisterType<SendGridEmailService>()
    .Keyed<IEmailService>("sendgrid");
```

---

## Ejercicios Prácticos

### Ejercicio 1: Identificar el Ciclo de Vida Apropiado

Determina el ciclo de vida (Transient, Scoped, Singleton) apropiado para cada servicio:

1. `IConfiguration` - Leer configuración de app.config
2. `IEmailSender` - Enviar emails sin estado
3. `IDbContext` - Contexto de Entity Framework
4. `IUserRepository` - Repositorio que usa IDbContext
5. `ILogger` - Servicio de logging thread-safe
6. `IHttpContextAccessor` - Acceso al contexto HTTP actual
7. `ICacheManager` - Caché en memoria compartida
8. `IOrderCalculator` - Calcular precios sin estado

**Respuestas**:
1. Singleton - La configuración no cambia
2. Transient - Sin estado, ligero
3. Scoped - Por petición HTTP
4. Scoped - Debe compartir el DbContext
5. Singleton - Thread-safe y compartido
6. Scoped - Diferente por petición
7. Singleton - Compartido entre todas las peticiones
8. Transient - Sin estado, puede ser diferente cada vez

### Ejercicio 2: Corregir Dependencias Incorrectas

Identifica y corrige el problema:

```csharp
public class ReportService
{
    private readonly IDbContext _context;
    
    public ReportService(IDbContext context)
    {
        _context = context;
    }
}

// Registro
services.AddSingleton<ReportService>();
services.AddScoped<IDbContext, MyDbContext>();
```

**Problema**: Captive Dependency - Singleton captura Scoped.

**Solución**:
```csharp
services.AddScoped<ReportService>(); // Cambiar a Scoped
services.AddScoped<IDbContext, MyDbContext>();
```

### Ejercicio 3: Implementar un Módulo de Autofac

Crea un módulo de Autofac que registre:
- Todos los repositorios (terminan en "Repository") como Scoped
- Todos los servicios (terminan en "Service") como Scoped
- ICacheService como Singleton
- IEmailService como Transient

**Solución**:

```csharp
public class ApplicationModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        // Repositorios
        builder.RegisterAssemblyTypes(ThisAssembly)
            .Where(t => t.Name.EndsWith("Repository"))
            .AsImplementedInterfaces()
            .InstancePerLifetimeScope();
        
        // Servicios
        builder.RegisterAssemblyTypes(ThisAssembly)
            .Where(t => t.Name.EndsWith("Service"))
            .AsImplementedInterfaces()
            .InstancePerLifetimeScope();
        
        // Caché Singleton
        builder.RegisterType<MemoryCacheService>()
            .As<ICacheService>()
            .SingleInstance();
        
        // Email Transient
        builder.RegisterType<SmtpEmailService>()
            .As<IEmailService>()
            .InstancePerDependency();
    }
}
```

### Ejercicio 4: Refactorizar Service Locator

Refactoriza este código que usa el anti-pattern Service Locator:

```csharp
public class OrderController : Controller
{
    public ActionResult Create(Order order)
    {
        var repository = DependencyResolver.Current.GetService<IOrderRepository>();
        repository.Add(order);
        
        var emailService = DependencyResolver.Current.GetService<IEmailService>();
        emailService.SendEmail(order.CustomerEmail, "Order Confirmed", "...");
        
        return RedirectToAction("Index");
    }
}
```

**Solución**:

```csharp
public class OrderController : Controller
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    
    public OrderController(IOrderRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
    
    public ActionResult Create(Order order)
    {
        _repository.Add(order);
        _emailService.SendEmail(order.CustomerEmail, "Order Confirmed", "...");
        
        return RedirectToAction("Index");
    }
}
```

### Ejercicio 5: Diseñar una Aplicación Testeable

Diseña la arquitectura de inyección de dependencias para una aplicación de e-commerce con:
- Controladores MVC
- Servicios de negocio
- Repositorios
- DbContext
- Servicios externos (Email, Payment Gateway)
- Caché

**Solución**:

```csharp
public class DependencyConfig
{
    public static void RegisterDependencies()
    {
        var container = new UnityContainer();
        
        // Data Access - Scoped
        container.RegisterType<ECommerceDbContext>(
            new HierarchicalLifetimeManager());
        
        // Repositories - Scoped (comparten DbContext)
        container.RegisterType<IProductRepository, ProductRepository>(
            new HierarchicalLifetimeManager());
        container.RegisterType<IOrderRepository, OrderRepository>(
            new HierarchicalLifetimeManager());
        container.RegisterType<ICustomerRepository, CustomerRepository>(
            new HierarchicalLifetimeManager());
        
        // Business Services - Scoped
        container.RegisterType<IProductService, ProductService>(
            new HierarchicalLifetimeManager());
        container.RegisterType<IOrderService, OrderService>(
            new HierarchicalLifetimeManager());
        container.RegisterType<ICustomerService, CustomerService>(
            new HierarchicalLifetimeManager());
        
        // External Services - Transient (stateless)
        container.RegisterType<IEmailService, SmtpEmailService>(
            new TransientLifetimeManager());
        container.RegisterType<IPaymentGateway, StripePaymentGateway>(
            new TransientLifetimeManager());
        
        // Infrastructure Services - Singleton
        container.RegisterType<ICacheService, MemoryCacheService>(
            new ContainerControlledLifetimeManager());
        container.RegisterType<IConfiguration, AppConfiguration>(
            new ContainerControlledLifetimeManager());
        container.RegisterType<ILogService, Log4NetService>(
            new ContainerControlledLifetimeManager());
        
        DependencyResolver.SetResolver(new UnityDependencyResolver(container));
    }
}
```

---

## Conclusión

Los contenedores IoC y la correcta gestión del ciclo de vida de dependencias son fundamentales para:

- **Crear código desacoplado y mantenible**
- **Facilitar las pruebas unitarias y de integración**
- **Mejorar el rendimiento de la aplicación**
- **Evitar memory leaks y problemas de concurrencia**
- **Seguir los principios SOLID**

### Puntos Clave para Recordar

1. **Transient**: Nueva instancia cada vez - servicios sin estado
2. **Scoped**: Una instancia por petición - repositorios y DbContext
3. **Singleton**: Una instancia para toda la app - configuración y caché
4. **Evitar Captive Dependencies**: No inyectar Scoped/Transient en Singleton
5. **Preferir Constructor Injection**: Hace las dependencias explícitas
6. **Thread-Safety en Singletons**: Deben ser thread-safe
7. **Disposable Pattern**: Implementar IDisposable cuando sea necesario

### Recursos Adicionales

- **Microsoft Dependency Injection**: https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection
- **Autofac Documentation**: https://autofac.readthedocs.io/
- **Unity Container**: https://github.com/unitycontainer/unity
- **Dependency Injection Principles, Practices, and Patterns** - Steven van Deursen & Mark Seemann

---


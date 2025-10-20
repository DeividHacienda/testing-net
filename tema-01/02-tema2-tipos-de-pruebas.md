# Tema 2: Tipos de pruebas: unitarias, de integración, funcionales, de UI, etc.

## Introducción

En el desarrollo de software moderno, las pruebas no son un concepto monolítico. Existen diferentes tipos de pruebas, cada una diseñada para validar aspectos específicos de nuestro sistema. Comprender cuándo y cómo aplicar cada tipo es fundamental para construir una estrategia de testing efectiva y equilibrada.

## La pirámide de pruebas

Antes de profundizar en cada tipo, es importante entender el concepto de la **pirámide de pruebas**, propuesto por Mike Cohn:

```
        /\
       /UI\
      /────\
     /Integr\
    /────────\
   /Unitarias \
  /────────────\
```

Esta pirámide nos indica:

- **Base amplia**: Mayor cantidad de pruebas unitarias (rápidas, baratas, aisladas)
- **Medio**: Cantidad moderada de pruebas de integración
- **Cima estrecha**: Pocas pruebas de UI/E2E (lentas, costosas, frágiles)

La razón de esta distribución es el **equilibrio entre costo y valor**:

| Tipo | Velocidad | Costo | Confiabilidad | Cobertura |
|------|-----------|-------|---------------|-----------|
| Unitarias | Muy rápida | Bajo | Alta | Baja |
| Integración | Media | Medio | Media | Media |
| UI/E2E | Lenta | Alto | Baja | Alta |

---

## 1. Pruebas Unitarias

### ¿Qué son?

Las pruebas unitarias verifican el comportamiento de **unidades individuales de código** (métodos, clases) de forma aislada, sin depender de componentes externos como bases de datos, servicios web o sistemas de archivos.

### Características principales

- **Rápidas**: Se ejecutan en milisegundos
- **Aisladas**: No dependen de BD, archivos, red, etc.
- **Deterministas**: Siempre producen el mismo resultado
- **Fáciles de mantener**: Un cambio pequeño afecta pocas pruebas
- **Independientes**: Cada prueba debe poder ejecutarse sola

### Patrón AAA (Arrange-Act-Assert)

Todas las pruebas unitarias siguen este patrón:

```csharp
[Fact]
public void NombreDelMetodo_Escenario_ResultadoEsperado()
{
    // Arrange (Preparar): Configurar objetos y datos necesarios
    var calculadora = new Calculadora();
    
    // Act (Actuar): Ejecutar el método bajo prueba
    var resultado = calculadora.Sumar(5, 3);
    
    // Assert (Afirmar): Verificar que el resultado es el esperado
    Assert.Equal(8, resultado);
}
```

### Ejemplo completo en .NET

```csharp
public class Calculadora
{
    public int Sumar(int a, int b) => a + b;
    
    public int Dividir(int dividendo, int divisor)
    {
        if (divisor == 0)
            throw new DivideByZeroException("No se puede dividir por cero");
        return dividendo / divisor;
    }
}

// Pruebas con xUnit
public class CalculadoraTests
{
    [Fact]
    public void Sumar_DosCantidadesPositivas_RetornaResultadoCorrecto()
    {
        // Arrange
        var calculadora = new Calculadora();
        
        // Act
        var resultado = calculadora.Sumar(5, 3);
        
        // Assert
        Assert.Equal(8, resultado);
    }
    
    [Theory]
    [InlineData(0, 0, 0)]
    [InlineData(-5, 5, 0)]
    [InlineData(-10, -5, -15)]
    [InlineData(100, 200, 300)]
    public void Sumar_VariosEscenarios_RetornaResultadoCorrecto(
        int a, int b, int esperado)
    {
        // Arrange
        var calculadora = new Calculadora();
        
        // Act
        var resultado = calculadora.Sumar(a, b);
        
        // Assert
        Assert.Equal(esperado, resultado);
    }
    
    [Fact]
    public void Dividir_DivisorCero_LanzaExcepcion()
    {
        // Arrange
        var calculadora = new Calculadora();
        
        // Act & Assert
        Assert.Throws<DivideByZeroException>(
            () => calculadora.Dividir(10, 0)
        );
    }
}
```

### ¿Qué probar en pruebas unitarias?

✅ **SÍ probar:**
- Lógica de negocio
- Validaciones y reglas
- Cálculos y transformaciones
- Condiciones límite (boundary conditions)
- Casos excepcionales
- Diferentes caminos de ejecución (branches)

❌ **NO probar:**
- Propiedades simples (getters/setters sin lógica)
- Código generado automáticamente
- Frameworks externos (ya están probados)
- Constructores triviales

### Ejemplo con lógica de negocio

```csharp
public class PedidoService
{
    public decimal CalcularTotal(Pedido pedido)
    {
        if (pedido == null)
            throw new ArgumentNullException(nameof(pedido));
            
        decimal subtotal = pedido.Items.Sum(i => i.Precio * i.Cantidad);
        decimal descuento = CalcularDescuento(subtotal, pedido.CodigoDescuento);
        decimal impuestos = (subtotal - descuento) * 0.21m;
        
        return subtotal - descuento + impuestos;
    }
    
    private decimal CalcularDescuento(decimal subtotal, string codigoDescuento)
    {
        return codigoDescuento switch
        {
            "DESC10" => subtotal * 0.10m,
            "DESC20" => subtotal * 0.20m,
            "DESC50" => subtotal * 0.50m,
            _ => 0
        };
    }
}

public class PedidoServiceTests
{
    [Fact]
    public void CalcularTotal_PedidoNulo_LanzaExcepcion()
    {
        var service = new PedidoService();
        Assert.Throws<ArgumentNullException>(() => service.CalcularTotal(null));
    }
    
    [Fact]
    public void CalcularTotal_SinDescuento_AplicaImpuestos()
    {
        // Arrange
        var service = new PedidoService();
        var pedido = new Pedido
        {
            Items = new List<PedidoItem>
            {
                new() { Precio = 100, Cantidad = 2 }
            }
        };
        
        // Act
        var total = service.CalcularTotal(pedido);
        
        // Assert
        Assert.Equal(242m, total); // 200 + (200 * 0.21)
    }
    
    [Theory]
    [InlineData("DESC10", 218m)]  // 200 - 20 + (180 * 0.21)
    [InlineData("DESC20", 193.6m)] // 200 - 40 + (160 * 0.21)
    [InlineData("INVALIDO", 242m)] // Sin descuento
    public void CalcularTotal_ConDescuento_AplicaDescuentoEImpuestos(
        string codigo, decimal esperado)
    {
        var service = new PedidoService();
        var pedido = new Pedido
        {
            Items = new List<PedidoItem>
            {
                new() { Precio = 100, Cantidad = 2 }
            },
            CodigoDescuento = codigo
        };
        
        var total = service.CalcularTotal(pedido);
        Assert.Equal(esperado, total);
    }
}
```

---

## 2. Pruebas de Integración

### ¿Qué son?

Las pruebas de integración verifican la **interacción entre múltiples componentes** del sistema, incluyendo bases de datos, APIs externas, sistemas de archivos, colas de mensajes, etc.

### Características principales

- **Más lentas** que las unitarias (segundos en lugar de milisegundos)
- **Requieren configuración** (BD de prueba, servicios mock)
- **Más frágiles** debido a dependencias externas
- **Mayor valor** al validar interacciones reales
- **Menor cantidad** que las unitarias

### Ejemplo con base de datos (Entity Framework Core)

```csharp
public class UsuarioRepository
{
    private readonly AppDbContext _context;
    
    public UsuarioRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<Usuario> GuardarAsync(Usuario usuario)
    {
        _context.Usuarios.Add(usuario);
        await _context.SaveChangesAsync();
        return usuario;
    }
    
    public async Task<Usuario> ObtenerPorEmailAsync(string email)
    {
        return await _context.Usuarios
            .FirstOrDefaultAsync(u => u.Email == email);
    }
}

// Fixture para compartir contexto de BD entre pruebas
public class DatabaseFixture : IDisposable
{
    private readonly string _connectionString;
    
    public DatabaseFixture()
    {
        var dbName = $"TestDB_{Guid.NewGuid()}";
        _connectionString = $"Server=(localdb)\\mssqllocaldb;Database={dbName};Integrated Security=true";
        
        using var context = CreateContext();
        context.Database.EnsureCreated();
    }
    
    public AppDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseSqlServer(_connectionString)
            .Options;
        return new AppDbContext(options);
    }
    
    public void Dispose()
    {
        using var context = CreateContext();
        context.Database.EnsureDeleted();
    }
}

// Pruebas de integración
public class UsuarioRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    
    public UsuarioRepositoryTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }
    
    [Fact]
    public async Task GuardarUsuario_UsuarioValido_SeGuardaCorrectamente()
    {
        // Arrange
        using var context = _fixture.CreateContext();
        var repository = new UsuarioRepository(context);
        var usuario = new Usuario 
        { 
            Nombre = "Juan Pérez", 
            Email = "juan@test.com" 
        };
        
        // Act
        await repository.GuardarAsync(usuario);
        
        // Assert
        var usuarioGuardado = await repository.ObtenerPorEmailAsync("juan@test.com");
        Assert.NotNull(usuarioGuardado);
        Assert.Equal("Juan Pérez", usuarioGuardado.Nombre);
        Assert.True(usuarioGuardado.Id > 0);
    }
    
    [Fact]
    public async Task ObtenerPorEmail_EmailNoExiste_RetornaNull()
    {
        // Arrange
        using var context = _fixture.CreateContext();
        var repository = new UsuarioRepository(context);
        
        // Act
        var usuario = await repository.ObtenerPorEmailAsync("noexiste@test.com");
        
        // Assert
        Assert.Null(usuario);
    }
}
```

### Pruebas con WebApplicationFactory (ASP.NET Core)

Para probar APIs completas:

```csharp
public class ProductosControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    
    public ProductosControllerTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task Get_TodosLosProductos_RetornaStatusOk()
    {
        // Act
        var response = await _client.GetAsync("/api/productos");
        
        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json; charset=utf-8", 
            response.Content.Headers.ContentType?.ToString());
    }
    
    [Fact]
    public async Task Post_ProductoValido_CreaProducto()
    {
        // Arrange
        var nuevoProducto = new
        {
            Nombre = "Laptop",
            Precio = 999.99m
        };
        var content = new StringContent(
            JsonSerializer.Serialize(nuevoProducto),
            Encoding.UTF8,
            "application/json"
        );
        
        // Act
        var response = await _client.PostAsync("/api/productos", content);
        
        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        var locationHeader = response.Headers.Location;
        Assert.NotNull(locationHeader);
    }
}
```

### Estrategias para bases de datos de prueba

1. **Base de datos en memoria (SQLite)**
   - Rápida
   - No requiere instalación
   - Limitaciones con características avanzadas

2. **Base de datos temporal**
   - Más realista
   - Más lenta
   - Requiere servidor SQL

3. **Testcontainers**
   - Contenedores Docker para pruebas
   - Aislamiento completo
   - Configuración realista

```csharp
// Ejemplo con SQLite en memoria
public class InMemoryDatabaseFixture : IDisposable
{
    public AppDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseSqlite("DataSource=:memory:")
            .Options;
            
        var context = new AppDbContext(options);
        context.Database.OpenConnection();
        context.Database.EnsureCreated();
        return context;
    }
    
    public void Dispose()
    {
        // Cleanup
    }
}
```

---

## 3. Pruebas Funcionales

### ¿Qué son?

Las pruebas funcionales validan que el sistema cumple con los **requisitos funcionales** especificados, probando funcionalidades completas desde la perspectiva del usuario, pero sin necesariamente interactuar con la UI.

### Diferencia con pruebas de integración

- **Integración**: Verifica que componentes técnicos funcionen juntos
- **Funcionales**: Verifica que el sistema cumpla requisitos de negocio

### Ejemplo: Proceso completo de compra

```csharp
public class ProcesoCompraTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public ProcesoCompraTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task ProcesoCompraCompleto_ClienteNuevo_CompraExitosa()
    {
        // 1. Crear usuario
        var usuario = await CrearUsuarioAsync("cliente@test.com", "Password123!");
        
        // 2. Iniciar sesión
        var token = await IniciarSesionAsync("cliente@test.com", "Password123!");
        _client.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", token);
        
        // 3. Agregar productos al carrito
        await AgregarProductoCarritoAsync(1, cantidad: 2);
        await AgregarProductoCarritoAsync(5, cantidad: 1);
        
        // 4. Obtener carrito
        var carrito = await ObtenerCarritoAsync();
        Assert.Equal(3, carrito.Items.Sum(i => i.Cantidad));
        
        // 5. Aplicar código de descuento
        await AplicarDescuentoAsync("DESC10");
        
        // 6. Procesar pago
        var pedido = await ProcesarPagoAsync(new DatosPago
        {
            NumeroTarjeta = "4111111111111111",
            Cvv = "123",
            FechaExpiracion = "12/25"
        });
        
        // 7. Verificar pedido creado
        Assert.NotNull(pedido);
        Assert.Equal("Confirmado", pedido.Estado);
        Assert.True(pedido.Total > 0);
        
        // 8. Verificar email de confirmación enviado
        var emails = await ObtenerEmailsEnviadosAsync(usuario.Email);
        Assert.Contains(emails, e => e.Asunto.Contains("Confirmación de pedido"));
    }
    
    private async Task<Usuario> CrearUsuarioAsync(string email, string password)
    {
        var content = new StringContent(
            JsonSerializer.Serialize(new { email, password }),
            Encoding.UTF8,
            "application/json"
        );
        var response = await _client.PostAsync("/api/usuarios/registro", content);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Usuario>();
    }
    
    // ... otros métodos auxiliares
}
```

### Características de buenas pruebas funcionales

1. **Prueban flujos completos** de usuario
2. **Independientes entre sí**
3. **Datos aislados** (cada prueba crea sus propios datos)
4. **Verifican resultados de negocio**, no detalles técnicos
5. **Legibles por no programadores** (idealmente)

---

## 4. Pruebas de Interfaz de Usuario (UI)

### ¿Qué son?

Las pruebas de UI automatizan la interacción con la interfaz gráfica del sistema, simulando acciones reales del usuario como clicks, escritura, navegación, etc.

### Características principales

- **Las más lentas**: Segundos o minutos por prueba
- **Las más frágiles**: Cambios en UI rompen pruebas
- **Mayor cobertura end-to-end**
- **Costosas de mantener**
- **Deben ser pocas y críticas**

### Herramientas en .NET

1. **Selenium**: Estándar de industria, multi-navegador
2. **Playwright**: Moderno, rápido, excelente API
3. **Puppeteer Sharp**: Control de Chrome/Chromium

### Ejemplo con Selenium

```csharp
public class LoginPageTests : IDisposable
{
    private readonly IWebDriver _driver;
    private readonly string _baseUrl = "https://localhost:5001";
    
    public LoginPageTests()
    {
        var options = new ChromeOptions();
        options.AddArgument("--headless"); // Sin interfaz gráfica
        _driver = new ChromeDriver(options);
        _driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
    }
    
    [Fact]
    public void Login_CredencialesValidas_RedirigeADashboard()
    {
        // Arrange
        _driver.Navigate().GoToUrl($"{_baseUrl}/login");
        
        // Act
        var emailInput = _driver.FindElement(By.Id("email"));
        var passwordInput = _driver.FindElement(By.Id("password"));
        var submitButton = _driver.FindElement(By.Id("btn-login"));
        
        emailInput.SendKeys("usuario@test.com");
        passwordInput.SendKeys("Password123!");
        submitButton.Click();
        
        // Assert
        var wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(5));
        wait.Until(d => d.Url.Contains("/dashboard"));
        
        Assert.Contains("/dashboard", _driver.Url);
        Assert.Contains("Bienvenido", _driver.PageSource);
    }
    
    [Fact]
    public void Login_CredencialesInvalidas_MuestraError()
    {
        // Arrange
        _driver.Navigate().GoToUrl($"{_baseUrl}/login");
        
        // Act
        _driver.FindElement(By.Id("email")).SendKeys("invalido@test.com");
        _driver.FindElement(By.Id("password")).SendKeys("wrongpass");
        _driver.FindElement(By.Id("btn-login")).Click();
        
        // Assert
        var errorMsg = _driver.FindElement(By.ClassName("error-message"));
        Assert.Contains("credenciales incorrectas", 
            errorMsg.Text, 
            StringComparison.OrdinalIgnoreCase);
    }
    
    public void Dispose()
    {
        _driver?.Quit();
        _driver?.Dispose();
    }
}
```

### Ejemplo con Playwright (más moderno)

```csharp
public class EcommerceTests : IAsyncLifetime
{
    private IPlaywright _playwright;
    private IBrowser _browser;
    private IPage _page;
    
    public async Task InitializeAsync()
    {
        _playwright = await Playwright.CreateAsync();
        _browser = await _playwright.Chromium.LaunchAsync(new()
        {
            Headless = true
        });
        _page = await _browser.NewPageAsync();
    }
    
    [Fact]
    public async Task CompraProducto_ProcesoCompleto_Exitoso()
    {
        // 1. Navegar a la página
        await _page.GotoAsync("https://localhost:5001");
        
        // 2. Buscar producto
        await _page.FillAsync("#search-input", "laptop");
        await _page.ClickAsync("#search-button");
        
        // 3. Esperar resultados
        await _page.WaitForSelectorAsync(".product-card");
        
        // 4. Seleccionar primer producto
        await _page.ClickAsync(".product-card:first-child");
        
        // 5. Agregar al carrito
        await _page.ClickAsync("#add-to-cart");
        
        // 6. Verificar notificación
        var notification = await _page.TextContentAsync(".notification");
        Assert.Contains("agregado al carrito", notification);
        
        // 7. Ir al carrito
        await _page.ClickAsync("#cart-icon");
        
        // 8. Verificar producto en carrito
        var cartItems = await _page.Locator(".cart-item").CountAsync();
        Assert.Equal(1, cartItems);
        
        // 9. Proceder al checkout
        await _page.ClickAsync("#checkout-button");
        
        // 10. Llenar datos de envío
        await _page.FillAsync("#nombre", "Juan Pérez");
        await _page.FillAsync("#direccion", "Calle Falsa 123");
        await _page.FillAsync("#ciudad", "Madrid");
        await _page.FillAsync("#codigo-postal", "28001");
        
        // 11. Continuar a pago
        await _page.ClickAsync("#continuar-pago");
        
        // 12. Llenar datos de pago
        await _page.FillAsync("#numero-tarjeta", "4111111111111111");
        await _page.FillAsync("#cvv", "123");
        await _page.SelectOptionAsync("#mes-expiracion", "12");
        await _page.SelectOptionAsync("#anio-expiracion", "2025");
        
        // 13. Confirmar pedido
        await _page.ClickAsync("#confirmar-pedido");
        
        // 14. Verificar confirmación
        await _page.WaitForSelectorAsync(".order-confirmation");
        var confirmacion = await _page.TextContentAsync("h1");
        Assert.Contains("Pedido confirmado", confirmacion);
        
        // 15. Verificar número de pedido
        var numeroPedido = await _page.TextContentAsync("#numero-pedido");
        Assert.Matches(@"#\d{6}", numeroPedido);
    }
    
    [Fact]
    public async Task FormularioContacto_DatosIncompletos_MuestraErrores()
    {
        await _page.GotoAsync("https://localhost:5001/contacto");
        
        // Enviar sin llenar campos
        await _page.ClickAsync("#enviar-formulario");
        
        // Verificar errores de validación
        var errores = await _page.Locator(".field-error").AllAsync();
        Assert.True(errores.Count >= 3); // Email, nombre, mensaje requeridos
    }
    
    public async Task DisposeAsync()
    {
        await _browser?.CloseAsync();
        _playwright?.Dispose();
    }
}
```

### Page Object Model (POM)

Patrón para organizar pruebas de UI de forma mantenible:

```csharp
public class LoginPage
{
    private readonly IPage _page;
    private readonly string _url;
    
    // Selectores centralizados
    private const string EmailInput = "#email";
    private const string PasswordInput = "#password";
    private const string LoginButton = "#btn-login";
    private const string ErrorMessage = ".error-message";
    
    public LoginPage(IPage page, string baseUrl)
    {
        _page = page;
        _url = $"{baseUrl}/login";
    }
    
    public async Task NavigateAsync()
    {
        await _page.GotoAsync(_url);
    }
    
    public async Task LoginAsync(string email, string password)
    {
        await _page.FillAsync(EmailInput, email);
        await _page.FillAsync(PasswordInput, password);
        await _page.ClickAsync(LoginButton);
    }
    
    public async Task<string> GetErrorMessageAsync()
    {
        return await _page.TextContentAsync(ErrorMessage);
    }
    
    public async Task<bool> IsErrorVisibleAsync()
    {
        return await _page.IsVisibleAsync(ErrorMessage);
    }
}

// Uso en tests
public class LoginTests
{
    [Fact]
    public async Task Login_CredencialesInvalidas_MuestraError()
    {
        var loginPage = new LoginPage(_page, "https://localhost:5001");
        
        await loginPage.NavigateAsync();
        await loginPage.LoginAsync("invalido@test.com", "wrongpass");
        
        Assert.True(await loginPage.IsErrorVisibleAsync());
        var error = await loginPage.GetErrorMessageAsync();
        Assert.Contains("incorrectas", error);
    }
}
```

### Mejores prácticas para pruebas de UI

1. **Usar Page Object Model** para mantenibilidad
2. **Esperas explícitas** en lugar de sleeps
3. **Selectores estables** (IDs, data-testid)
4. **Datos de prueba aislados**
5. **Screenshots en fallos**
6. **Ejecución en paralelo** cuando sea posible
7. **Mínima cantidad**: Solo flujos críticos

```csharp
// Configuración de screenshots en fallos
[Fact]
public async Task Test_ConCapturaDePantalla()
{
    try
    {
        // ... código de prueba
    }
    catch (Exception)
    {
        var timestamp = DateTime.Now.ToString("yyyyMMdd_HHmmss");
        await _page.ScreenshotAsync(new()
        {
            Path = $"screenshots/error_{timestamp}.png",
            FullPage = true
        });
        throw;
    }
}
```

---

## 5. Pruebas de Rendimiento (Performance)

### ¿Qué son?

Verifican que el sistema mantiene tiempos de respuesta aceptables bajo diferentes condiciones de carga.

### Tipos de pruebas de rendimiento

1. **Load Testing**: Comportamiento bajo carga normal/esperada
2. **Stress Testing**: Límites del sistema bajo carga extrema
3. **Spike Testing**: Respuesta ante picos súbitos de carga
4. **Endurance Testing**: Comportamiento durante períodos prolongados

### Ejemplo con BenchmarkDotNet

```csharp
[MemoryDiagnoser]
[RankColumn]
public class AlgoritmosBenchmark
{
    private List<int> _data;
    
    [GlobalSetup]
    public void Setup()
    {
        _data = Enumerable.Range(1, 10000).ToList();
    }
    
    [Benchmark(Baseline = true)]
    public int ForLoop()
    {
        int sum = 0;
        for (int i = 0; i < _data.Count; i++)
        {
            sum += _data[i];
        }
        return sum;
    }
    
    [Benchmark]
    public int ForEachLoop()
    {
        int sum = 0;
        foreach (var item in _data)
        {
            sum += item;
        }
        return sum;
    }
    
    [Benchmark]
    public int LinqSum()
    {
        return _data.Sum();
    }
}

// Ejecución
public class Program
{
    public static void Main(string[] args)
    {
        var summary = BenchmarkRunner.Run<AlgoritmosBenchmark>();
    }
}
```

---

## 6. Pruebas de Seguridad

### ¿Qué verifican?

Vulnerabilidades y problemas de seguridad en la aplicación.

### Ejemplos comunes

```csharp
public class SeguridadTests
{
    [Fact]
    public async Task Api_SinAutenticacion_RetornaUnauthorized()
    {
        var client = new HttpClient();
        var response = await client.GetAsync("https://api.example.com/datos-privados");
        
        Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
    }
    
    [Theory]
    [InlineData("'; DROP TABLE Users--")]
    [InlineData("<script>alert('XSS')</script>")]
    [InlineData("../../../etc/passwd")]
    public async Task Formulario_EntradaMaliciosa_EsSanitizada(string input)
    {
        // Verificar que inputs maliciosos son sanitizados/rechazados
        var response = await _client.PostAsync("/api/comentarios", new
        {
            texto = input
        });
        
        // No debe devolver 500 (error no manejado)
        Assert.NotEqual(HttpStatusCode.InternalServerError, response.StatusCode);
    }
}
```

---

## 7. Comparación y Cuándo Usar Cada Tipo

### Tabla comparativa

| Aspecto | Unitarias | Integración | Funcionales | UI | Rendimiento |
|---------|-----------|-------------|-------------|-----|-------------|
| **Velocidad** | ⚡⚡⚡ Muy rápida | ⚡⚡ Media | ⚡ Lenta | 🐌 Muy lenta | ⚡ Variable |
| **Costo mantener** | 💰 Bajo | 💰💰 Medio | 💰💰 Medio | 💰💰💰 Alto | 💰💰 Medio |
| **Estabilidad** | ✅✅✅ Alta | ✅✅ Media | ✅✅ Media | ⚠️ Baja | ✅ Media |
| **Cobertura** | 🎯 Específica | 🎯🎯 Media | 🎯🎯🎯 Amplia | 🎯🎯🎯 E2E | 🎯 Específica |
| **Cantidad** | 1000s | 100s | 10s | <10 | Variable |

### Estrategia recomendada

```
Proyecto pequeño (< 10k líneas):
├── 80% Unitarias (~200-500 tests)
├── 15% Integración (~30-80 tests)
└── 5% UI (~5-10 tests)

Proyecto mediano (10k-100k líneas):
├── 70% Unitarias (~2000-5000 tests)
├── 20% Integración (~500-1500 tests)
├── 8% Funcionales (~100-400 tests)
└── 2% UI (~20-50 tests)

Proyecto grande (>100k líneas):
├── 60% Unitarias (~10000+ tests)
├── 25% Integración (~4000+ tests)
├── 12% Funcionales (~2000+ tests)
└── 3% UI (~500+ tests)
```

### Criterios de decisión

**Usa pruebas UNITARIAS cuando:**
- ✅ Pruebes lógica de negocio aislada
- ✅ Necesites retroalimentación rápida
- ✅ Quieras documentar comportamiento de métodos
- ✅ Practiques TDD

**Usa pruebas de INTEGRACIÓN cuando:**
- ✅ Verifiques interacción con BD
- ✅ Pruebes llamadas a APIs externas
- ✅ Valides configuración de ORM
- ✅ Necesites probar transacciones

**Usa pruebas FUNCIONALES cuando:**
- ✅ Verifiques requisitos de negocio completos
- ✅ Pruebes flujos de usuario end-to-end
- ✅ Necesites validar múltiples componentes juntos
- ✅ Documentes casos de uso

**Usa pruebas de UI cuando:**
- ✅ Verifiques flujos críticos de negocio
- ✅ Pruebes funcionalidad que requiere UI específica
- ✅ Valides compatibilidad entre navegadores
- ✅ Necesites pruebas de regresión visuales

**Usa pruebas de RENDIMIENTO cuando:**
- ✅ Tengas SLAs de tiempo de respuesta
- ✅ Optimices código crítico
- ✅ Prepares para lanzamientos
- ✅ Detectes degradación de performance

---

## 8. Anti-patrones a Evitar

### ❌ El cono de helado invertido

```
Muchas pruebas UI (lentas, frágiles)
Pocas pruebas de integración
Muy pocas pruebas unitarias
```

**Problema**: Feedback lento, tests frágiles, alto costo de mantenimiento

### ❌ Pruebas que prueban frameworks

```csharp
// ❌ MAL: Probando Entity Framework, no tu lógica
[Fact]
public void DbContext_GuardarCambios_Funciona()
{
    var context = new AppDbContext();
    var entity = new Usuario { Nombre = "Test" };
    context.Add(entity);
    context.SaveChanges();
    
    Assert.NotEqual(0, entity.Id);
}
```

### ❌ Pruebas dependientes entre sí

```csharp
// ❌ MAL: Test2 depende que Test1 se ejecute primero
[Fact]
public void Test1_CreaUsuario()
{
    var usuario = new Usuario { Id = 1, Nombre = "Juan" };
    _service.Guardar(usuario);
}

[Fact]
public void Test2_ActualizaUsuario() // ⚠️ Requiere Test1
{
    var usuario = _service.ObtenerPorId(1);
    usuario.Nombre = "Pedro";
    _service.Actualizar(usuario);
}
```

**Solución**: Cada test debe crear sus propios datos

### ❌ Tests con sleeps/delays

```csharp
// ❌ MAL: Tests lentos e impredecibles
await Task.Delay(5000); // Esperar 5 segundos
Assert.True(_element.IsVisible);
```

**Solución**: Usar esperas explícitas con condiciones

```csharp
// ✅ BIEN
var wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(10));
wait.Until(d => d.FindElement(By.Id("elemento")).Displayed);
```

---

## 9. Resumen y Conclusiones

### Puntos clave

1. **No existe un tipo de prueba perfecto**: Cada uno tiene su propósito
2. **La pirámide de pruebas** es una guía, no una regla absoluta
3. **Equilibrio**: Busca el balance entre cobertura, velocidad y mantenibilidad
4. **Empieza simple**: Comienza con unitarias, añade otros tipos según necesites
5. **Calidad sobre cantidad**: 10 buenos tests valen más que 100 malos

### Checklist de tipos de pruebas

Para cualquier proyecto .NET profesional:

- [ ] ✅ Pruebas unitarias para lógica de negocio
- [ ] ✅ Pruebas de integración para BD y APIs
- [ ] ✅ Al menos 2-3 pruebas funcionales de flujos críticos
- [ ] ✅ 1-2 pruebas de UI para flujos más importantes
- [ ] [ ] Pruebas de rendimiento (si hay SLAs)
- [ ] [ ] Pruebas de seguridad (en proyectos expuestos públicamente)



---

## Referencias y recursos adicionales

- [Microsoft Docs - Unit testing in .NET](https://docs.microsoft.com/dotnet/core/testing/)
- [Martin Fowler - Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)
- [xUnit Documentation](https://xunit.net/)
- [Playwright for .NET](https://playwright.dev/dotnet/)
- [BenchmarkDotNet](https://benchmarkdotnet.org/)

---



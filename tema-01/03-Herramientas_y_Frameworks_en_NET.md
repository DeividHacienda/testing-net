# Herramientas y frameworks en .NET

## Teoría

### ¿Qué son los frameworks de testing?

Los frameworks de testing son bibliotecas y herramientas que proporcionan la infraestructura necesaria para escribir, organizar y ejecutar pruebas de software de manera sistemática. En el ecosistema .NET, existen múltiples frameworks especializados para diferentes tipos de pruebas, cada uno con sus características, ventajas y casos de uso específicos.

Un framework de testing típicamente proporciona:

**Estructura organizativa:** Permite agrupar pruebas en clases, métodos y suites, facilitando su mantenimiento y ejecución selectiva.

**Assertions:** Métodos para verificar que los resultados obtenidos coinciden con los esperados, proporcionando mensajes de error claros cuando fallan.

**Test Runners:** Motores de ejecución que descubren, ejecutan y reportan los resultados de las pruebas, integrándose con IDEs y sistemas de CI/CD.

**Fixtures y Setup:** Mecanismos para preparar el entorno antes de las pruebas y limpiarlo después, garantizando que cada prueba se ejecuta en un estado conocido.

**Extensibilidad:** Puntos de extensión para personalizar el comportamiento del framework según las necesidades del proyecto.

### Categorías principales de herramientas

El ecosistema de testing en .NET se puede clasificar en varias categorías según su propósito:

#### 1. Frameworks de pruebas unitarias

Proporcionan la base para escribir y ejecutar pruebas unitarias. Incluyen atributos para marcar métodos de prueba, assertions para validar resultados, y runners para ejecutarlas. Los principales son MSTest, xUnit y NUnit.

**Características comunes:**
- Descubrimiento automático de pruebas mediante atributos
- Sistema de assertions rico y expresivo
- Soporte para setup y teardown de pruebas
- Integración con Visual Studio Test Explorer
- Compatibilidad con .NET Framework 4.x

#### 2. Frameworks de mocking

Facilitan la creación de objetos simulados (mocks, stubs, fakes) que reemplazan dependencias reales durante las pruebas. Permiten aislar el código bajo prueba y verificar interacciones entre componentes.

**Capacidades principales:**
- Creación dinámica de mocks sin escribir clases manualmente
- Configuración de comportamientos esperados
- Verificación de llamadas a métodos
- Simulación de excepciones y casos extremos

Los más utilizados son Moq, NSubstitute y FakeItEasy.

#### 3. Herramientas de cobertura de código

Miden qué porcentaje del código es ejecutado por las pruebas, identificando áreas sin cobertura. Esto ayuda a detectar código no probado y evaluar la efectividad del conjunto de pruebas.

**Métricas que proporcionan:**
- Cobertura de líneas
- Cobertura de ramas (branches)
- Cobertura de métodos
- Cobertura de clases

Las herramientas principales incluyen Coverlet, dotCover y OpenCover.

#### 4. Frameworks de pruebas de integración

Facilitan pruebas que involucran múltiples componentes trabajando juntos, como bases de datos, APIs externas o sistemas de archivos. En .NET Framework, se utilizan técnicas como Self-Hosting de Web API, Microsoft Fakes y enfoques de integración tradicionales.

**Características:**
- Levantamiento de entornos de prueba
- Gestión de bases de datos de prueba
- Simulación de servicios externos
- Transacciones automáticas con rollback

#### 5. Frameworks de pruebas de UI

Automatizan interacciones con interfaces de usuario, simulando acciones de usuarios reales como clics, escritura en campos y navegación.

**Capacidades:**
- Control de navegadores reales o headless
- Localización de elementos del DOM
- Espera inteligente de elementos
- Captura de screenshots y videos

Selenium y Playwright son las opciones más populares.

#### 6. Herramientas de análisis estático

Analizan el código sin ejecutarlo, detectando problemas de calidad, vulnerabilidades de seguridad y violaciones de estándares de codificación.

**Aspectos que analizan:**
- Complejidad ciclomática
- Code smells
- Vulnerabilidades de seguridad
- Cumplimiento de estándares

SonarQube, Roslyn Analyzers y StyleCop son ejemplos destacados.

#### 7. Frameworks de testing de comportamiento (BDD)

Permiten escribir pruebas en lenguaje natural que documentan el comportamiento esperado del sistema desde la perspectiva del negocio.

**Ventajas:**
- Colaboración entre técnicos y no técnicos
- Documentación ejecutable del comportamiento
- Enfoque en el valor de negocio

SpecFlow es el framework BDD más utilizado en .NET.

---

## Comparativa de Frameworks de Pruebas Unitarias

### MSTest

**Descripción:**
MSTest es el framework de testing oficial de Microsoft, integrado nativamente en Visual Studio. Es la opción tradicional para .NET Framework y continúa siendo ampliamente utilizado en aplicaciones empresariales.

**Ventajas:**
- Integración perfecta con Visual Studio sin configuración adicional
- Soporte oficial de Microsoft con actualizaciones regulares
- Documentación extensa y recursos de aprendizaje abundantes
- Curva de aprendizaje suave para desarrolladores .NET
- Soporte completo para DataRow y pruebas parametrizadas

**Desventajas:**
- Sintaxis más verbosa comparada con xUnit
- Menos características avanzadas que xUnit
- Comunidad más pequeña que xUnit
- Menos extensible que sus alternativas

**Casos de uso ideales:**
- Equipos que trabajan exclusivamente en el ecosistema Microsoft
- Proyectos empresariales que requieren soporte oficial
- Desarrolladores nuevos en testing que buscan la opción más simple
- Migración desde proyectos legacy que ya usan MSTest

**Ejemplo:**

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class CalculadoraTests
{
    private Calculadora _calculadora;
    
    [TestInitialize]
    public void Setup()
    {
        _calculadora = new Calculadora();
    }
    
    [TestMethod]
    public void Sumar_DosNumeros_RetornaResultadoCorrecto()
    {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int resultado = _calculadora.Sumar(a, b);
        
        // Assert
        Assert.AreEqual(8, resultado);
    }
    
    [TestMethod]
    [DataRow(2, 3, 5)]
    [DataRow(0, 0, 0)]
    [DataRow(-1, 1, 0)]
    public void Sumar_VariosValores_RetornaResultadoCorrecto(int a, int b, int esperado)
    {
        int resultado = _calculadora.Sumar(a, b);
        Assert.AreEqual(esperado, resultado);
    }
    
    [TestCleanup]
    public void Cleanup()
    {
        _calculadora = null;
    }
}
```

### xUnit

**Descripción:**
xUnit es un framework moderno creado por los autores originales de NUnit, diseñado con lecciones aprendidas de frameworks anteriores. Aunque es popular en .NET Core, también es totalmente compatible con .NET Framework y ofrece características modernas para proyectos tradicionales.

**Ventajas:**
- Diseño moderno con mejores prácticas incorporadas
- Cada prueba se ejecuta en una instancia nueva de la clase (mejor aislamiento)
- No requiere atributos para setup/teardown, usa el constructor y Dispose
- Excelente soporte para pruebas asíncronas
- Extensibilidad superior mediante atributos personalizados
- Sintaxis más limpia y expresiva
- Compatible con .NET Framework 4.5.2 y superior

**Desventajas:**
- Curva de aprendizaje ligeramente mayor para principiantes
- Algunos conceptos (como la ausencia de [TestInitialize]) pueden confundir inicialmente
- Documentación menos abundante que MSTest para escenarios enterprise

**Casos de uso ideales:**
- Proyectos .NET Framework modernos que buscan mejores prácticas
- Equipos que valoran código limpio y moderno
- Migración gradual hacia arquitecturas más modernas
- Aplicaciones con fuerte énfasis en pruebas asíncronas
- Equipos experimentados en testing

**Ejemplo:**

```csharp
using Xunit;

public class CalculadoraTests : IDisposable
{
    private readonly Calculadora _calculadora;
    
    // Constructor actúa como Setup
    public CalculadoraTests()
    {
        _calculadora = new Calculadora();
    }
    
    [Fact]
    public void Sumar_DosNumeros_RetornaResultadoCorrecto()
    {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int resultado = _calculadora.Sumar(a, b);
        
        // Assert
        Assert.Equal(8, resultado);
    }
    
    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(0, 0, 0)]
    [InlineData(-1, 1, 0)]
    public void Sumar_VariosValores_RetornaResultadoCorrecto(int a, int b, int esperado)
    {
        int resultado = _calculadora.Sumar(a, b);
        Assert.Equal(esperado, resultado);
    }
    
    // Dispose actúa como Cleanup
    public void Dispose()
    {
        // Limpieza de recursos si es necesario
    }
}
```

### NUnit

**Descripción:**
NUnit es uno de los frameworks de testing más antiguos y maduros del ecosistema .NET, portado originalmente desde JUnit. Ha evolucionado a lo largo de muchos años y cuenta con una amplia base de usuarios.

**Ventajas:**
- Framework muy maduro con años de evolución
- Sintaxis rica y expresiva para assertions
- Excelente soporte para pruebas parametrizadas con múltiples fuentes de datos
- Características avanzadas como TestCase con argumentos complejos
- Constraints muy potentes y legibles
- Gran cantidad de atributos y opciones de configuración
- Comunidad grande y activa

**Desventajas:**
- Puede resultar más complejo debido a la cantidad de opciones
- Sintaxis de assertions diferente al estándar de xUnit
- Configuración inicial ligeramente más compleja
- Menos adoptado en proyectos nuevos comparado con xUnit

**Casos de uso ideales:**
- Proyectos legacy que ya utilizan NUnit
- Equipos que requieren assertions muy expresivas
- Pruebas con múltiples fuentes de datos complejas
- Proyectos que valoran la madurez y estabilidad sobre la modernidad

**Ejemplo:**

```csharp
using NUnit.Framework;

[TestFixture]
public class CalculadoraTests
{
    private Calculadora _calculadora;
    
    [SetUp]
    public void Setup()
    {
        _calculadora = new Calculadora();
    }
    
    [Test]
    public void Sumar_DosNumeros_RetornaResultadoCorrecto()
    {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int resultado = _calculadora.Sumar(a, b);
        
        // Assert
        Assert.That(resultado, Is.EqualTo(8));
    }
    
    [TestCase(2, 3, 5)]
    [TestCase(0, 0, 0)]
    [TestCase(-1, 1, 0)]
    public void Sumar_VariosValores_RetornaResultadoCorrecto(int a, int b, int esperado)
    {
        int resultado = _calculadora.Sumar(a, b);
        Assert.That(resultado, Is.EqualTo(esperado));
    }
    
    [Test]
    public void Dividir_PorCero_LanzaExcepcion()
    {
        Assert.That(() => _calculadora.Dividir(10, 0), 
                    Throws.TypeOf<DivideByZeroException>());
    }
    
    [TearDown]
    public void Cleanup()
    {
        _calculadora = null;
    }
}
```

### Tabla Comparativa de Frameworks Unitarios

| Característica | MSTest | xUnit | NUnit |
|----------------|--------|-------|-------|
| **Mantenedor** | Microsoft | Comunidad (.NET Foundation) | Comunidad |
| **Sintaxis** | Tradicional | Moderna y limpia | Rica y expresiva |
| **Setup/Teardown** | [TestInitialize]/[TestCleanup] | Constructor/Dispose | [SetUp]/[TearDown] |
| **Aislamiento** | Instancia compartida | Nueva instancia por test | Instancia compartida |
| **Pruebas parametrizadas** | [DataRow] | [Theory]/[InlineData] | [TestCase] |
| **Assertions** | Assert.AreEqual | Assert.Equal | Assert.That con Constraints |
| **Curva de aprendizaje** | Baja | Media | Media-Alta |
| **Compatibilidad .NET Framework** | Completa | 4.5.2+ | Completa |
| **Extensibilidad** | Limitada | Excelente | Muy Buena |
| **Integración VS** | Nativa | Excelente | Excelente |
| **Documentación** | Extensa | Buena | Extensa |
| **Comunidad** | Grande | Muy Grande | Grande |
| **Mejor para** | Enterprise tradicional | Proyectos modernos | Pruebas complejas |

---

## Comparativa de Frameworks de Mocking

### Moq

**Descripción:**
Moq es el framework de mocking más popular en .NET, conocido por su sintaxis fluida y expresiva basada en expresiones lambda. Es la opción predeterminada para la mayoría de proyectos .NET.

**Ventajas:**
- Sintaxis muy intuitiva y legible
- API fluida basada en lambdas
- Ampliamente adoptado con gran cantidad de recursos
- Verificación potente de llamadas a métodos
- Soporte completo para propiedades, eventos y métodos genéricos
- Excelente integración con xUnit, NUnit y MSTest
- Comunidad muy activa

**Desventajas:**
- Curva de aprendizaje inicial para entender la sintaxis de Setup
- Algunos escenarios complejos requieren configuración verbosa
- Mensajes de error pueden ser crípticos en casos avanzados

**Casos de uso ideales:**
- La mayoría de proyectos .NET (opción estándar)
- Equipos que valoran sintaxis expresiva
- Proyectos con pruebas unitarias extensivas
- Cuando se necesita verificación detallada de interacciones

**Ejemplo:**

```csharp
using Moq;
using Xunit;

public interface IRepositorioUsuarios
{
    Usuario ObtenerPorId(int id);
    void Guardar(Usuario usuario);
    bool Existe(string email);
}

public class ServicioUsuariosTests
{
    [Fact]
    public void RegistrarUsuario_UsuarioNuevo_GuardaEnRepositorio()
    {
        // Arrange
        var mockRepositorio = new Mock<IRepositorioUsuarios>();
        mockRepositorio.Setup(r => r.Existe(It.IsAny<string>())).Returns(false);
        
        var servicio = new ServicioUsuarios(mockRepositorio.Object);
        var nuevoUsuario = new Usuario { Email = "test@test.com", Nombre = "Test" };
        
        // Act
        servicio.RegistrarUsuario(nuevoUsuario);
        
        // Assert
        mockRepositorio.Verify(r => r.Guardar(It.Is<Usuario>(u => u.Email == "test@test.com")), 
                               Times.Once());
    }
    
    [Fact]
    public void ObtenerUsuario_UsuarioExiste_RetornaUsuario()
    {
        // Arrange
        var usuarioEsperado = new Usuario { Id = 1, Nombre = "Juan" };
        var mockRepositorio = new Mock<IRepositorioUsuarios>();
        mockRepositorio.Setup(r => r.ObtenerPorId(1)).Returns(usuarioEsperado);
        
        var servicio = new ServicioUsuarios(mockRepositorio.Object);
        
        // Act
        var resultado = servicio.ObtenerUsuario(1);
        
        // Assert
        Assert.Equal(usuarioEsperado.Nombre, resultado.Nombre);
        mockRepositorio.Verify(r => r.ObtenerPorId(1), Times.Once());
    }
}
```

### NSubstitute

**Descripción:**
NSubstitute es un framework de mocking diseñado para ser amigable y minimalista, con una sintaxis extremadamente limpia que reduce el ruido en las pruebas.

**Ventajas:**
- Sintaxis la más limpia y concisa de todos
- Curva de aprendizaje muy suave
- No requiere diferencia explícita entre stubs y mocks
- Configuración y verificación con la misma sintaxis
- Excelente para equipos nuevos en testing
- Mensajes de error claros y útiles

**Desventajas:**
- Menos adoptado que Moq
- Comunidad más pequeña
- Menos ejemplos y recursos disponibles
- Algunas características avanzadas menos desarrolladas

**Casos de uso ideales:**
- Equipos que priorizan simplicidad sobre funcionalidades avanzadas
- Proyectos donde la legibilidad es crítica
- Desarrolladores nuevos en mocking
- Pruebas donde el foco está en el comportamiento más que en la verificación detallada

**Ejemplo:**

```csharp
using NSubstitute;
using Xunit;

public class ServicioUsuariosTests
{
    [Fact]
    public void RegistrarUsuario_UsuarioNuevo_GuardaEnRepositorio()
    {
        // Arrange
        var repositorio = Substitute.For<IRepositorioUsuarios>();
        repositorio.Existe(Arg.Any<string>()).Returns(false);
        
        var servicio = new ServicioUsuarios(repositorio);
        var nuevoUsuario = new Usuario { Email = "test@test.com", Nombre = "Test" };
        
        // Act
        servicio.RegistrarUsuario(nuevoUsuario);
        
        // Assert
        repositorio.Received(1).Guardar(Arg.Is<Usuario>(u => u.Email == "test@test.com"));
    }
    
    [Fact]
    public void ObtenerUsuario_UsuarioExiste_RetornaUsuario()
    {
        // Arrange
        var usuarioEsperado = new Usuario { Id = 1, Nombre = "Juan" };
        var repositorio = Substitute.For<IRepositorioUsuarios>();
        repositorio.ObtenerPorId(1).Returns(usuarioEsperado);
        
        var servicio = new ServicioUsuarios(repositorio);
        
        // Act
        var resultado = servicio.ObtenerUsuario(1);
        
        // Assert
        Assert.Equal(usuarioEsperado.Nombre, resultado.Nombre);
        repositorio.Received(1).ObtenerPorId(1);
    }
}
```

### FakeItEasy

**Descripción:**
FakeItEasy es un framework de mocking que busca el equilibrio entre potencia y facilidad de uso, con una API intuitiva y mensajes de error excepcionales.

**Ventajas:**
- Terminología simple (todo son "fakes")
- No necesitas distinguir entre mocks, stubs y dummies
- Mensajes de error excelentes y muy descriptivos
- API consistente y predecible
- Buena documentación
- Sintaxis natural y fácil de leer

**Desventajas:**
- Menos popular que Moq y NSubstitute
- Comunidad más pequeña
- Menos integración en tutoriales y cursos
- Algunos desarrolladores prefieren la distinción explícita de tipos de test doubles

**Casos de uso ideales:**
- Equipos que valoran mensajes de error claros
- Proyectos donde se busca consistencia en la API
- Desarrolladores que se confunden con la terminología tradicional de mocking
- Cuando la facilidad de debugging es prioritaria

**Ejemplo:**

```csharp
using FakeItEasy;
using Xunit;

public class ServicioUsuariosTests
{
    [Fact]
    public void RegistrarUsuario_UsuarioNuevo_GuardaEnRepositorio()
    {
        // Arrange
        var repositorio = A.Fake<IRepositorioUsuarios>();
        A.CallTo(() => repositorio.Existe(A<string>._)).Returns(false);
        
        var servicio = new ServicioUsuarios(repositorio);
        var nuevoUsuario = new Usuario { Email = "test@test.com", Nombre = "Test" };
        
        // Act
        servicio.RegistrarUsuario(nuevoUsuario);
        
        // Assert
        A.CallTo(() => repositorio.Guardar(A<Usuario>.That.Matches(u => u.Email == "test@test.com")))
            .MustHaveHappenedOnceExactly();
    }
    
    [Fact]
    public void ObtenerUsuario_UsuarioExiste_RetornaUsuario()
    {
        // Arrange
        var usuarioEsperado = new Usuario { Id = 1, Nombre = "Juan" };
        var repositorio = A.Fake<IRepositorioUsuarios>();
        A.CallTo(() => repositorio.ObtenerPorId(1)).Returns(usuarioEsperado);
        
        var servicio = new ServicioUsuarios(repositorio);
        
        // Act
        var resultado = servicio.ObtenerUsuario(1);
        
        // Assert
        Assert.Equal(usuarioEsperado.Nombre, resultado.Nombre);
        A.CallTo(() => repositorio.ObtenerPorId(1)).MustHaveHappenedOnceExactly();
    }
}
```

### Tabla Comparativa de Frameworks de Mocking

| Característica | Moq | NSubstitute | FakeItEasy |
|----------------|-----|-------------|------------|
| **Popularidad** | Muy Alta | Media | Media-Baja |
| **Sintaxis** | Fluida con lambdas | Muy limpia | Natural y consistente |
| **Curva de aprendizaje** | Media | Baja | Baja |
| **Setup** | .Setup() | Directa | A.CallTo() |
| **Verificación** | .Verify() | .Received() | .MustHaveHappened() |
| **Mensajes de error** | Buenos | Muy Buenos | Excelentes |
| **Comunidad** | Muy Grande | Media | Pequeña |
| **Documentación** | Extensa | Buena | Buena |
| **Recursos disponibles** | Abundantes | Moderados | Limitados |
| **Extensibilidad** | Excelente | Buena | Buena |
| **Mejor para** | Uso general | Simplicidad | Claridad de errores |

---

## Herramientas de Cobertura de Código

### OpenCover

**Descripción:**
OpenCover es una herramienta open source de cobertura de código específicamente diseñada para .NET Framework. Es la opción más utilizada y madura para medir la cobertura en proyectos .NET Framework tradicionales.

**Características principales:**
- Gratuito y open source
- Soporte completo para .NET Framework 2.0 a 4.8
- Múltiples formatos de salida (OpenCover XML, compatible con herramientas de reporting)
- Integración con herramientas de CI/CD
- Compatible con todos los frameworks de testing (MSTest, NUnit, xUnit)

**Instalación:**

```bash
# Instalar mediante NuGet Package Manager
Install-Package OpenCover

# O mediante línea de comandos
nuget install OpenCover
```

**Uso básico:**

```bash
# Ejecutar pruebas con OpenCover
OpenCover.Console.exe ^
  -target:"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\MSTest.exe" ^
  -targetargs:"/testcontainer:.\MiProyecto.Tests\bin\Debug\MiProyecto.Tests.dll" ^
  -filter:"+[MiProyecto*]* -[MiProyecto.Tests*]*" ^
  -register:user ^
  -output:coverage.xml

# Para NUnit
OpenCover.Console.exe ^
  -target:"nunit3-console.exe" ^
  -targetargs:"MiProyecto.Tests.dll" ^
  -filter:"+[MiProyecto*]* -[*Tests]*" ^
  -output:coverage.xml

# Para xUnit
OpenCover.Console.exe ^
  -target:"xunit.console.exe" ^
  -targetargs:"MiProyecto.Tests.dll -noshadow" ^
  -filter:"+[MiProyecto*]* -[*Tests]*" ^
  -output:coverage.xml
```

**Filtros de cobertura:**

```bash
# Incluir todos los ensamblados excepto tests
-filter:"+[*]* -[*.Tests]* -[*TestHelpers]*"

# Incluir solo ensamblados específicos
-filter:"+[MiProyecto.Core]* +[MiProyecto.Services]* -[*.Tests]*"

# Excluir clases específicas
-filter:"+[MiProyecto*]* -[MiProyecto]*Properties.* -[MiProyecto]*Program"
```

**Generación de reportes HTML con ReportGenerator:**

```bash
# Instalar ReportGenerator
nuget install ReportGenerator

# Generar reporte HTML desde el XML de OpenCover
ReportGenerator.exe ^
  -reports:coverage.xml ^
  -targetdir:coveragereport ^
  -reporttypes:Html

# Abrir reporte
start coveragereport\index.html
```

**Integración en archivo .bat para automatización:**

```batch
@echo off
echo Ejecutando pruebas con cobertura...

REM Limpiar resultados anteriores
if exist coverage.xml del coverage.xml
if exist coveragereport rmdir /s /q coveragereport

REM Ejecutar OpenCover
packages\OpenCover.4.7.1221\tools\OpenCover.Console.exe ^
  -target:"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\MSTest.exe" ^
  -targetargs:"/testcontainer:.\MiProyecto.Tests\bin\Debug\MiProyecto.Tests.dll" ^
  -filter:"+[MiProyecto*]* -[MiProyecto.Tests*]*" ^
  -register:user ^
  -output:coverage.xml

REM Generar reporte HTML
packages\ReportGenerator.5.1.10\tools\net47\ReportGenerator.exe ^
  -reports:coverage.xml ^
  -targetdir:coveragereport ^
  -reporttypes:Html;Badges

echo Reporte generado en: coveragereport\index.html
start coveragereport\index.html
```

### dotCover (JetBrains)

**Descripción:**
dotCover es una herramienta comercial de JetBrains integrada en ReSharper y disponible como herramienta independiente. Ofrece la mejor experiencia de usuario para .NET Framework con integración profunda en Visual Studio.

**Características principales:**
- Interfaz gráfica intuitiva directamente en Visual Studio
- Análisis de cobertura en tiempo real mientras escribes código
- Highlighting de código cubierto y no cubierto en el editor
- Detección de código duplicado
- Filtros avanzados y perfiles de cobertura
- Snapshots de cobertura para comparación histórica
- Integración con TeamCity y herramientas JetBrains

**Ventajas:**
- Mejor experiencia de usuario del mercado
- Visualización instantánea en el editor
- No requiere configuración de línea de comandos
- Análisis profundo de gaps de cobertura
- Soporte técnico profesional de JetBrains
- Compatible con todos los frameworks de testing

**Desventajas:**
- Requiere licencia comercial (aproximadamente $149 individual, incluida con ReSharper Ultimate)
- Solo disponible para Windows
- Mayor consumo de recursos que herramientas de línea de comandos

**Uso desde Visual Studio:**

1. Instalar dotCover (viene con ReSharper o como herramienta independiente)
2. Clic derecho en proyecto de tests > "Cover Unit Tests"
3. Ver resultados directamente en el editor con highlighting
4. Exportar reportes en múltiples formatos

**Uso desde línea de comandos:**

```bash
# Ejecutar cobertura con dotCover CLI
dotCover.exe cover ^
  /TargetExecutable="C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\MSTest.exe" ^
  /TargetArguments="/testcontainer:MiProyecto.Tests.dll" ^
  /Output=coverage.dcvr ^
  /ReportType=HTML ^
  /Filters="+:MiProyecto.*;-:*.Tests"
```

### Visual Studio Enterprise Code Coverage

**Descripción:**
Visual Studio Enterprise incluye herramientas nativas de cobertura de código sin necesidad de instalación adicional. Es la opción más integrada para equipos que ya tienen licencias Enterprise.

**Características:**
- Integración nativa en Visual Studio Enterprise
- No requiere herramientas externas
- Interfaz visual en el IDE
- Highlighting de cobertura en el código
- Exportación a formato .coverage

**Uso:**

```
1. Test > Analyze Code Coverage > All Tests
2. Ver resultados en ventana "Code Coverage Results"
3. El código se colorea automáticamente (azul = cubierto, rojo = no cubierto)
```

**Conversión a formato XML para reportes:**

```bash
# Usar CodeCoverage.exe (incluido en VS Enterprise)
CodeCoverage.exe analyze /output:coverage.coveragexml TestResults\*.coverage
```

### Coverlet (Opcional para .NET Framework)

**Nota:** Aunque Coverlet está diseñado principalmente para .NET Core, puede usarse con proyectos .NET Framework 4.6.1+ que utilicen el nuevo formato de .csproj basado en SDK.

**Instalación solo si tu proyecto usa SDK-style csproj:**

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="coverlet.collector" Version="6.0.0">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

**Uso:**

```bash
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
```

---

## Herramientas de Análisis Estático

### SonarQube / SonarCloud

**Descripción:**
SonarQube es una plataforma completa de calidad de código que realiza análisis estático continuo, detectando bugs, vulnerabilidades, code smells y midiendo la deuda técnica.

**Características principales:**
- Análisis de múltiples lenguajes incluido C#
- Detección de vulnerabilidades de seguridad
- Medición de deuda técnica
- Quality Gates personalizables
- Integración con CI/CD pipelines
- Dashboard centralizado para equipos

**Reglas que analiza:**
- Bugs potenciales
- Vulnerabilidades de seguridad (OWASP Top 10)
- Code smells y anti-patrones
- Complejidad ciclomática
- Duplicación de código
- Cobertura de pruebas

**Configuración básica:**

```yaml
# sonar-project.properties
sonar.projectKey=mi-proyecto
sonar.projectName=Mi Proyecto .NET
sonar.projectVersion=1.0
sonar.sources=src
sonar.tests=tests
sonar.cs.opencover.reportsPaths=coverage.opencover.xml
sonar.exclusions=**/bin/**,**/obj/**
```

**Integración en pipeline CI/CD:**

```bash
# Instalar SonarScanner
dotnet tool install --global dotnet-sonarscanner

# Iniciar análisis
dotnet sonarscanner begin /k:"proyecto" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="token"

# Compilar proyecto
dotnet build

# Ejecutar pruebas con cobertura
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover

# Finalizar análisis
dotnet sonarscanner end /d:sonar.login="token"
```

### Roslyn Analyzers

**Descripción:**
Los Roslyn Analyzers son analizadores basados en el compilador de C# que proporcionan análisis en tiempo real durante la escritura de código en Visual Studio.

**Tipos de analyzers:**

**Analyzers de estilo:**
- Convenciones de nomenclatura
- Formateo de código
- Uso de this y var

**Analyzers de calidad:**
- Detección de código no usado
- Parámetros no utilizados
- Expresiones simplificables

**Analyzers de seguridad:**
- SQL Injection
- XSS vulnerabilities
- Uso inseguro de criptografía

**Configuración mediante .editorconfig:**

```ini
# .editorconfig
root = true

[*.cs]
# Severity levels: none, silent, suggestion, warning, error
dotnet_diagnostic.CA1001.severity = warning
dotnet_diagnostic.CA1031.severity = suggestion
dotnet_diagnostic.IDE0005.severity = warning

# Naming conventions
dotnet_naming_rule.interfaces_should_be_prefixed_with_i.severity = warning
dotnet_naming_rule.interfaces_should_be_prefixed_with_i.symbols = interface
dotnet_naming_rule.interfaces_should_be_prefixed_with_i.style = begins_with_i

# Code style rules
csharp_prefer_braces = true:warning
csharp_using_directive_placement = outside_namespace:warning
```

**Paquetes NuGet recomendados:**

```xml
<ItemGroup>
  <!-- Analyzers de código -->
  <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" Version="8.0.0">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  </PackageReference>
  
  <!-- Analyzers de seguridad -->
  <PackageReference Include="Microsoft.CodeAnalysis.BannedApiAnalyzers" Version="3.3.4">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  </PackageReference>
  
  <!-- StyleCop para convenciones de estilo -->
  <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.507">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  </PackageReference>
</ItemGroup>
```

### StyleCop

**Descripción:**
StyleCop analiza el código fuente C# para hacer cumplir un conjunto de reglas de estilo y consistencia. Ayuda a mantener código limpio, legible y mantenible en todo el equipo.

**Categorías de reglas:**
- Ordenación de elementos
- Documentación XML
- Layout y espaciado
- Legibilidad
- Nomenclatura

**Configuración en stylecop.json:**

```json
{
  "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
  "settings": {
    "documentationRules": {
      "companyName": "Mi Empresa",
      "copyrightText": "Copyright (c) {companyName}. Todos los derechos reservados.",
      "xmlHeader": false,
      "documentPrivateElements": false
    },
    "orderingRules": {
      "usingDirectivesPlacement": "outsideNamespace",
      "elementOrder": [
        "kind",
        "accessibility",
        "constant",
        "static",
        "readonly"
      ]
    },
    "namingRules": {
      "allowCommonHungarianPrefixes": false,
      "allowedHungarianPrefixes": []
    }
  }
}
```

---

## Frameworks de Pruebas de Integración

### Pruebas de Integración en .NET Framework

**Descripción:**
Las pruebas de integración en .NET Framework requieren enfoques diferentes según el tipo de aplicación. Para Web API y MVC, se utilizan técnicas como Self-Hosting, mientras que para aplicaciones tradicionales se trabaja con bases de datos de prueba y contenedores de IoC.

### Self-Hosting para ASP.NET Web API

**Descripción:**
El Self-Hosting permite ejecutar un servidor Web API en memoria dentro de las pruebas, sin necesidad de IIS o servidor externo.

**Instalación:**

```bash
Install-Package Microsoft.AspNet.WebApi.SelfHost
Install-Package Microsoft.AspNet.WebApi.Client
```

**Ejemplo de Self-Hosting:**

```csharp
using System.Web.Http;
using System.Web.Http.SelfHost;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.Net.Http;

[TestClass]
public class UsuariosApiIntegrationTests
{
    private HttpSelfHostServer _server;
    private HttpClient _client;
    private string _baseAddress = "http://localhost:8080";

    [TestInitialize]
    public void Setup()
    {
        // Configurar servidor self-hosted
        var config = new HttpSelfHostConfiguration(_baseAddress);
        
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );

        _server = new HttpSelfHostServer(config);
        _server.OpenAsync().Wait();
        
        _client = new HttpClient { BaseAddress = new Uri(_baseAddress) };
    }

    [TestMethod]
    public async Task ObtenerUsuarios_RetornaStatusOk()
    {
        // Act
        var response = await _client.GetAsync("/api/usuarios");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.AreEqual("application/json", 
                       response.Content.Headers.ContentType.MediaType);
    }

    [TestMethod]
    public async Task CrearUsuario_ConDatosValidos_RetornaCreated()
    {
        // Arrange
        var nuevoUsuario = new { Nombre = "Test", Email = "test@test.com" };
        var content = new StringContent(
            JsonConvert.SerializeObject(nuevoUsuario),
            Encoding.UTF8,
            "application/json");

        // Act
        var response = await _client.PostAsync("/api/usuarios", content);

        // Assert
        Assert.AreEqual(HttpStatusCode.Created, response.StatusCode);
    }

    [TestCleanup]
    public void Cleanup()
    {
        _client?.Dispose();
        _server?.CloseAsync().Wait();
        _server?.Dispose();
    }
}
```

### OWIN Self-Hosting (Alternativa moderna para .NET Framework)

**Descripción:**
OWIN proporciona una abstracción entre servidores web y aplicaciones, permitiendo self-hosting más flexible.

**Instalación:**

```bash
Install-Package Microsoft.Owin.Hosting
Install-Package Microsoft.Owin.Host.HttpListener
Install-Package Microsoft.AspNet.WebApi.Owin
```

**Ejemplo con OWIN:**

```csharp
using Microsoft.Owin.Testing;
using Owin;
using System.Web.Http;
using Xunit;

public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        var config = new HttpConfiguration();
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
        
        app.UseWebApi(config);
    }
}

public class UsuariosApiTests
{
    [Fact]
    public async Task ObtenerUsuarios_RetornaOk()
    {
        // Arrange
        using (var server = TestServer.Create<Startup>())
        {
            // Act
            var response = await server.HttpClient.GetAsync("/api/usuarios");

            // Assert
            response.EnsureSuccessStatusCode();
        }
    }
}
```

### Pruebas de Integración con Base de Datos

**Descripción:**
Para probar la capa de acceso a datos y repositorios, es fundamental usar bases de datos de prueba que se puedan limpiar fácilmente.

**Enfoque 1: SQL Server LocalDB**

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.Data.SqlClient;

[TestClass]
public class UsuarioRepositoryIntegrationTests
{
    private string _connectionString = 
        @"Data Source=(LocalDB)\MSSQLLocalDB;Initial Catalog=TestDB;Integrated Security=True";
    private IUsuarioRepository _repository;

    [TestInitialize]
    public void Setup()
    {
        // Crear base de datos de prueba
        CrearBaseDatos();
        _repository = new UsuarioRepository(_connectionString);
    }

    [TestMethod]
    public void GuardarUsuario_EnBaseDeDatos_PersisteDatos()
    {
        // Arrange
        var usuario = new Usuario 
        { 
            Nombre = "Juan Pérez", 
            Email = "juan@test.com" 
        };

        // Act
        _repository.Guardar(usuario);
        var usuarioGuardado = _repository.ObtenerPorEmail("juan@test.com");

        // Assert
        Assert.IsNotNull(usuarioGuardado);
        Assert.AreEqual("Juan Pérez", usuarioGuardado.Nombre);
    }

    [TestCleanup]
    public void Cleanup()
    {
        // Eliminar base de datos de prueba
        EliminarBaseDatos();
    }

    private void CrearBaseDatos()
    {
        var masterConnection = 
            @"Data Source=(LocalDB)\MSSQLLocalDB;Initial Catalog=master;Integrated Security=True";
        
        using (var connection = new SqlConnection(masterConnection))
        {
            connection.Open();
            var command = connection.CreateCommand();
            command.CommandText = @"
                IF EXISTS (SELECT name FROM sys.databases WHERE name = 'TestDB')
                    DROP DATABASE TestDB;
                CREATE DATABASE TestDB;";
            command.ExecuteNonQuery();
        }

        // Crear esquema
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var command = connection.CreateCommand();
            command.CommandText = @"
                CREATE TABLE Usuarios (
                    Id INT IDENTITY PRIMARY KEY,
                    Nombre NVARCHAR(100) NOT NULL,
                    Email NVARCHAR(100) NOT NULL
                );";
            command.ExecuteNonQuery();
        }
    }

    private void EliminarBaseDatos()
    {
        var masterConnection = 
            @"Data Source=(LocalDB)\MSSQLLocalDB;Initial Catalog=master;Integrated Security=True";
        
        using (var connection = new SqlConnection(masterConnection))
        {
            connection.Open();
            var command = connection.CreateCommand();
            command.CommandText = @"
                IF EXISTS (SELECT name FROM sys.databases WHERE name = 'TestDB')
                    DROP DATABASE TestDB;";
            command.ExecuteNonQuery();
        }
    }
}
```

**Enfoque 2: Transacciones con Rollback**

```csharp
using System.Transactions;

[TestClass]
public class UsuarioRepositoryTransactionTests
{
    private TransactionScope _transaction;
    private IUsuarioRepository _repository;

    [TestInitialize]
    public void Setup()
    {
        // Iniciar transacción que se revertirá después de cada prueba
        _transaction = new TransactionScope();
        _repository = new UsuarioRepository(connectionString);
    }

    [TestMethod]
    public void GuardarUsuario_DentroDeTransaccion_SeReverteAlFinal()
    {
        // Arrange
        var usuario = new Usuario { Nombre = "Test", Email = "test@test.com" };

        // Act
        _repository.Guardar(usuario);
        var usuarioGuardado = _repository.ObtenerPorEmail("test@test.com");

        // Assert
        Assert.IsNotNull(usuarioGuardado);
        // La transacción se revertirá en Cleanup, no quedará en DB
    }

    [TestCleanup]
    public void Cleanup()
    {
        // Revertir transacción automáticamente
        _transaction.Dispose();
    }
}
```

**Enfoque 3: Entity Framework con InMemory (EF6)**

```csharp
using System.Data.Entity;
using Effort;

[TestClass]
public class UsuarioRepositoryEFTests
{
    private ApplicationDbContext _context;
    private IUsuarioRepository _repository;

    [TestInitialize]
    public void Setup()
    {
        // Usar Effort para base de datos en memoria
        var connection = DbConnectionFactory.CreateTransient();
        _context = new ApplicationDbContext(connection);
        _repository = new UsuarioRepository(_context);
    }

    [TestMethod]
    public void GuardarUsuario_ConEntityFramework_PersisteDatos()
    {
        // Arrange
        var usuario = new Usuario { Nombre = "Test", Email = "test@test.com" };

        // Act
        _repository.Guardar(usuario);
        _context.SaveChanges();
        
        var usuarioGuardado = _repository.ObtenerPorEmail("test@test.com");

        // Assert
        Assert.IsNotNull(usuarioGuardado);
        Assert.AreEqual("Test", usuarioGuardado.Nombre);
    }

    [TestCleanup]
    public void Cleanup()
    {
        _context.Dispose();
    }
}
```

### Microsoft Fakes (Visual Studio Enterprise)

**Descripción:**
Microsoft Fakes permite crear shims y stubs para aislar componentes durante las pruebas de integración, especialmente útil para código legacy difícil de modificar.

**Características:**
- Shims: Redirigen llamadas a métodos estáticos o sellados
- Stubs: Implementaciones falsas de interfaces
- Solo disponible en Visual Studio Enterprise

**Ejemplo con Shims:**

```csharp
using Microsoft.QualityTools.Testing.Fakes;
using System.Fakes;

[TestMethod]
public void ProcesarPedido_ConFechaActual_UsaFechaCorrecta()
{
    using (ShimsContext.Create())
    {
        // Arrange
        var fechaFija = new DateTime(2024, 1, 15);
        System.Fakes.ShimDateTime.NowGet = () => fechaFija;

        var servicio = new ServicioPedidos();

        // Act
        var pedido = servicio.CrearPedido();

        // Assert
        Assert.AreEqual(fechaFija, pedido.FechaCreacion);
    }
}
```

### Respawn (Limpieza de Base de Datos)

**Descripción:**
Respawn es una librería que permite resetear bases de datos entre pruebas de forma eficiente, eliminando todos los datos sin borrar el esquema.

**Instalación:**

```bash
Install-Package Respawn
```

**Uso:**

```csharp
using Respawn;

[TestClass]
public class IntegrationTestBase
{
    private static Checkpoint _checkpoint;
    protected string ConnectionString = 
        @"Server=(localdb)\mssqllocaldb;Database=TestDB;Integrated Security=true";

    [AssemblyInitialize]
    public static void AssemblyInit(TestContext context)
    {
        _checkpoint = new Checkpoint
        {
            TablesToIgnore = new[] { "__MigrationHistory" },
            SchemasToInclude = new[] { "dbo" }
        };
    }

    [TestInitialize]
    public async Task ResetDatabase()
    {
        await _checkpoint.Reset(ConnectionString);
    }
}

[TestClass]
public class UsuarioTests : IntegrationTestBase
{
    [TestMethod]
    public void PruebaConBaseDeDatosLimpia()
    {
        // La base de datos está limpia gracias a Respawn
        // pero el esquema sigue intacto
    }
}
```
            });

            // Reemplazar servicios con mocks
            services.AddScoped<IEmailService, MockEmailService>();

            // Inicializar base de datos de prueba
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            db.Database.EnsureCreated();
            SeedDatabase(db);
        });
    }

    private void SeedDatabase(ApplicationDbContext db)
    {
        db.Usuarios.Add(new Usuario { Id = 1, Nombre = "Test User" });
        db.SaveChanges();
    }
}
```

---

## Frameworks de Pruebas de UI

### Selenium WebDriver

**Descripción:**
Selenium es el framework líder en automatización de navegadores web, soportando todos los navegadores principales y múltiples lenguajes de programación incluido C#.

**Características:**
- Soporte para Chrome, Firefox, Edge, Safari
- API consistente entre navegadores
- Ejecución en modo headless
- Captura de screenshots y videos
- Grid para ejecución paralela

**Instalación:**

```bash
dotnet add package Selenium.WebDriver
dotnet add package Selenium.WebDriver.ChromeDriver
```

**Ejemplo básico:**

```csharp
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Support.UI;
using Xunit;

public class LoginTests : IDisposable
{
    private readonly IWebDriver _driver;
    private readonly WebDriverWait _wait;

    public LoginTests()
    {
        var options = new ChromeOptions();
        options.AddArgument("--headless"); // Ejecutar sin GUI
        options.AddArgument("--no-sandbox");
        
        _driver = new ChromeDriver(options);
        _wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(10));
    }

    [Fact]
    public void Login_ConCredencialesValidas_RedireccionaADashboard()
    {
        // Arrange
        _driver.Navigate().GoToUrl("https://ejemplo.com/login");

        // Act
        var emailInput = _driver.FindElement(By.Id("email"));
        var passwordInput = _driver.FindElement(By.Id("password"));
        var loginButton = _driver.FindElement(By.Id("login-button"));

        emailInput.SendKeys("usuario@test.com");
        passwordInput.SendKeys("password123");
        loginButton.Click();

        // Assert
        _wait.Until(d => d.Url.Contains("/dashboard"));
        Assert.Contains("dashboard", _driver.Url);
    }

    [Fact]
    public void Login_ConCredencialesInvalidas_MuestraError()
    {
        // Arrange
        _driver.Navigate().GoToUrl("https://ejemplo.com/login");

        // Act
        _driver.FindElement(By.Id("email")).SendKeys("invalido@test.com");
        _driver.FindElement(By.Id("password")).SendKeys("wrongpassword");
        _driver.FindElement(By.Id("login-button")).Click();

        // Assert
        var errorMessage = _wait.Until(d => 
            d.FindElement(By.ClassName("error-message")));
        Assert.Contains("Credenciales inválidas", errorMessage.Text);
    }

    public void Dispose()
    {
        _driver.Quit();
        _driver.Dispose();
    }
}
```

**Page Object Pattern con Selenium:**

```csharp
public class LoginPage
{
    private readonly IWebDriver _driver;
    private readonly WebDriverWait _wait;

    private IWebElement EmailInput => _driver.FindElement(By.Id("email"));
    private IWebElement PasswordInput => _driver.FindElement(By.Id("password"));
    private IWebElement LoginButton => _driver.FindElement(By.Id("login-button"));
    private IWebElement ErrorMessage => _driver.FindElement(By.ClassName("error-message"));

    public LoginPage(IWebDriver driver)
    {
        _driver = driver;
        _wait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));
    }

    public void Navigate()
    {
        _driver.Navigate().GoToUrl("https://ejemplo.com/login");
    }

    public void Login(string email, string password)
    {
        EmailInput.SendKeys(email);
        PasswordInput.SendKeys(password);
        LoginButton.Click();
    }

    public string GetErrorMessage()
    {
        _wait.Until(d => ErrorMessage.Displayed);
        return ErrorMessage.Text;
    }

    public bool IsOnDashboard()
    {
        _wait.Until(d => d.Url.Contains("/dashboard"));
        return _driver.Url.Contains("/dashboard");
    }
}

// Uso en pruebas
public class LoginTestsWithPOM
{
    private readonly IWebDriver _driver;
    private readonly LoginPage _loginPage;

    public LoginTestsWithPOM()
    {
        _driver = new ChromeDriver();
        _loginPage = new LoginPage(_driver);
    }

    [Fact]
    public void Login_ConCredencialesValidas_RedireccionaADashboard()
    {
        _loginPage.Navigate();
        _loginPage.Login("usuario@test.com", "password123");
        Assert.True(_loginPage.IsOnDashboard());
    }
}
```

### Playwright

**Descripción:**
Playwright es un framework moderno de Microsoft para automatización de navegadores, diseñado para aplicaciones web modernas con enfoque en confiabilidad y velocidad.

**Ventajas sobre Selenium:**
- Espera automática de elementos (auto-waiting)
- Captura de contextos de navegador
- Interceptación de red
- Mejor manejo de Shadow DOM
- Ejecución más rápida y estable
- API más moderna y consistente

**Instalación:**

```bash
dotnet add package Microsoft.Playwright
# Instalar navegadores
pwsh bin/Debug/net8.0/playwright.ps1 install
```

**Ejemplo básico:**

```csharp
using Microsoft.Playwright;
using Xunit;

public class LoginTestsPlaywright : IAsyncLifetime
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
    public async Task Login_ConCredencialesValidas_RedireccionaADashboard()
    {
        // Arrange
        await _page.GotoAsync("https://ejemplo.com/login");

        // Act
        await _page.FillAsync("#email", "usuario@test.com");
        await _page.FillAsync("#password", "password123");
        await _page.ClickAsync("#login-button");

        // Assert - Espera automática hasta que URL cambie
        await _page.WaitForURLAsync("**/dashboard");
        Assert.Contains("dashboard", _page.Url);
    }

    [Fact]
    public async Task Login_ConCredencialesInvalidas_MuestraError()
    {
        // Arrange
        await _page.GotoAsync("https://ejemplo.com/login");

        // Act
        await _page.FillAsync("#email", "invalido@test.com");
        await _page.FillAsync("#password", "wrongpassword");
        await _page.ClickAsync("#login-button");

        // Assert - Espera automática del elemento
        var errorText = await _page.TextContentAsync(".error-message");
        Assert.Contains("Credenciales inválidas", errorText);
    }

    [Fact]
    public async Task CapturaScreenshot_CuandoPruebaFalla()
    {
        await _page.GotoAsync("https://ejemplo.com");
        await _page.ScreenshotAsync(new()
        {
            Path = "screenshot.png",
            FullPage = true
        });
    }

    public async Task DisposeAsync()
    {
        await _browser.CloseAsync();
        _playwright.Dispose();
    }
}
```

**Características avanzadas de Playwright:**

```csharp
public class PlaywrightAdvancedTests
{
    [Fact]
    public async Task InterceptarPeticionesRed()
    {
        var playwright = await Playwright.CreateAsync();
        var browser = await playwright.Chromium.LaunchAsync();
        var page = await browser.NewPageAsync();

        // Interceptar y modificar respuestas
        await page.RouteAsync("**/api/usuarios", async route =>
        {
            var response = await route.FetchAsync();
            var json = await response.JsonAsync();
            // Modificar respuesta
            await route.FulfillAsync(new()
            {
                Response = response,
                Json = new { data = "modificado" }
            });
        });

        await page.GotoAsync("https://ejemplo.com");
    }

    [Fact]
    public async Task SimularDispositvosMoviles()
    {
        var playwright = await Playwright.CreateAsync();
        var browser = await playwright.Chromium.LaunchAsync();
        
        // Emular iPhone 12
        var context = await browser.NewContextAsync(playwright.Devices["iPhone 12"]);
        var page = await context.NewPageAsync();
        
        await page.GotoAsync("https://ejemplo.com");
        
        // La página se renderiza como en iPhone 12
    }

    [Fact]
    public async Task PruebasConAutenticacion()
    {
        var playwright = await Playwright.CreateAsync();
        var browser = await playwright.Chromium.LaunchAsync();
        
        // Guardar estado de autenticación
        var context = await browser.NewContextAsync();
        var page = await context.NewPageAsync();
        
        await page.GotoAsync("https://ejemplo.com/login");
        await page.FillAsync("#email", "usuario@test.com");
        await page.FillAsync("#password", "password123");
        await page.ClickAsync("#login-button");
        
        // Guardar cookies y storage
        await context.StorageStateAsync(new() { Path = "auth.json" });
        
        // Reutilizar en otras pruebas
        var newContext = await browser.NewContextAsync(new() 
        { 
            StorageStatePath = "auth.json" 
        });
        var newPage = await newContext.NewPageAsync();
        await newPage.GotoAsync("https://ejemplo.com/dashboard");
        // Ya está autenticado
    }
}
```

### Comparativa Selenium vs Playwright

| Característica | Selenium | Playwright |
|----------------|----------|------------|
| **Mantenedor** | Comunidad Open Source | Microsoft |
| **Madurez** | Muy maduro (desde 2004) | Moderno (desde 2020) |
| **Navegadores** | Chrome, Firefox, Edge, Safari | Chrome, Firefox, WebKit |
| **Espera automática** | Manual (WebDriverWait) | Automática |
| **Velocidad** | Media | Alta |
| **Estabilidad** | Requiere configuración | Muy estable por defecto |
| **API** | Tradicional | Moderna y async/await |
| **Interceptación de red** | Limitada | Completa |
| **Screenshots/Videos** | Básico | Avanzado |
| **Curva de aprendizaje** | Media | Baja |
| **Comunidad** | Muy grande | Creciente |
| **Documentación** | Extensa | Excelente |
| **Mejor para** | Proyectos legacy | Proyectos nuevos |

---

## Framework BDD: SpecFlow

**Descripción:**
SpecFlow es el framework de Behavior-Driven Development (BDD) más popular para .NET, permitiendo escribir pruebas en lenguaje natural (Gherkin) que son comprensibles tanto para técnicos como para stakeholders del negocio.

**Conceptos clave:**

**Feature Files:** Archivos .feature que contienen especificaciones en lenguaje Gherkin
**Step Definitions:** Implementaciones en C# de los pasos descritos en Gherkin
**Scenarios:** Casos de prueba específicos
**Background:** Pasos comunes a todos los escenarios de una feature

**Instalación:**

```bash
dotnet add package SpecFlow
dotnet add package SpecFlow.xUnit
dotnet add package SpecFlow.Tools.MsBuild.Generation
```

**Ejemplo de Feature File:**

```gherkin
# features/Login.feature
Feature: Autenticación de usuarios
  Como usuario del sistema
  Quiero poder iniciar sesión
  Para acceder a mi cuenta

  Background:
    Given que la aplicación está en funcionamiento
    And existe un usuario con email "usuario@test.com" y contraseña "Password123"

  Scenario: Login exitoso con credenciales válidas
    Given que estoy en la página de login
    When ingreso "usuario@test.com" en el campo email
    And ingreso "Password123" en el campo contraseña
    And hago clic en el botón de login
    Then debo ser redirigido al dashboard
    And debo ver un mensaje de bienvenida

  Scenario: Login fallido con contraseña incorrecta
    Given que estoy en la página de login
    When ingreso "usuario@test.com" en el campo email
    And ingreso "PasswordIncorrecta" en el campo contraseña
    And hago clic en el botón de login
    Then debo permanecer en la página de login
    And debo ver el mensaje de error "Credenciales inválidas"

  Scenario Outline: Login con diferentes credenciales inválidas
    Given que estoy en la página de login
    When ingreso "<email>" en el campo email
    And ingreso "<contraseña>" en el campo contraseña
    And hago clic en el botón de login
    Then debo ver el mensaje de error "<mensaje>"

    Examples:
      | email              | contraseña   | mensaje                    |
      |                    | Password123  | Email es requerido         |
      | usuario@test.com   |              | Contraseña es requerida    |
      | emailinvalido      | Password123  | Formato de email inválido  |
```

**Step Definitions:**

```csharp
using TechTalk.SpecFlow;
using Xunit;

[Binding]
public class LoginSteps
{
    private readonly ScenarioContext _scenarioContext;
    private readonly IWebDriver _driver;
    private LoginPage _loginPage;

    public LoginSteps(ScenarioContext scenarioContext)
    {
        _scenarioContext = scenarioContext;
        _driver = new ChromeDriver();
        _loginPage = new LoginPage(_driver);
    }

    [Given(@"que la aplicación está en funcionamiento")]
    public void GivenQueAplicacionEstaFuncionando()
    {
        // Verificar que la aplicación responde
        var response = _driver.Navigate().GoToUrl("https://ejemplo.com/health");
        Assert.Equal(200, response.StatusCode);
    }

    [Given(@"existe un usuario con email ""(.*)"" y contraseña ""(.*)""")]
    public void GivenExisteUsuario(string email, string password)
    {
        // Crear usuario en base de datos de prueba
        var userService = _scenarioContext.Get<IUserService>();
        userService.CreateUser(email, password);
    }

    [Given(@"que estoy en la página de login")]
    public void GivenEstoyEnPaginaLogin()
    {
        _loginPage.Navigate();
    }

    [When(@"ingreso ""(.*)"" en el campo email")]
    public void WhenIngresoEmail(string email)
    {
        _loginPage.EnterEmail(email);
    }

    [When(@"ingreso ""(.*)"" en el campo contraseña")]
    public void WhenIngresoPassword(string password)
    {
        _loginPage.EnterPassword(password);
    }

    [When(@"hago clic en el botón de login")]
    public void WhenHagoClicBotonLogin()
    {
        _loginPage.ClickLoginButton();
    }

    [Then(@"debo ser redirigido al dashboard")]
    public void ThenDeboSerRedirigidoDashboard()
    {
        Assert.Contains("dashboard", _driver.Url);
    }

    [Then(@"debo ver un mensaje de bienvenida")]
    public void ThenDeboVerMensajeBienvenida()
    {
        var welcomeMessage = _driver.FindElement(By.ClassName("welcome-message"));
        Assert.True(welcomeMessage.Displayed);
    }

    [Then(@"debo permanecer en la página de login")]
    public void ThenDeboPermanecerPaginaLogin()
    {
        Assert.Contains("login", _driver.Url);
    }

    [Then(@"debo ver el mensaje de error ""(.*)""")]
    public void ThenDeboVerMensajeError(string expectedMessage)
    {
        var errorMessage = _driver.FindElement(By.ClassName("error-message")).Text;
        Assert.Contains(expectedMessage, errorMessage);
    }

    [AfterScenario]
    public void Cleanup()
    {
        _driver.Quit();
        _driver.Dispose();
    }
}
```

**Hooks y contexto compartido:**

```csharp
[Binding]
public class Hooks
{
    private readonly ScenarioContext _scenarioContext;

    public Hooks(ScenarioContext scenarioContext)
    {
        _scenarioContext = scenarioContext;
    }

    [BeforeScenario]
    public void BeforeScenario()
    {
        // Inicializar base de datos de prueba
        var dbContext = new TestDbContext();
        dbContext.Database.EnsureCreated();
        _scenarioContext.Set(dbContext);

        // Configurar servicios
        var userService = new UserService(dbContext);
        _scenarioContext.Set<IUserService>(userService);
    }

    [AfterScenario]
    public void AfterScenario()
    {
        // Limpiar base de datos
        var dbContext = _scenarioContext.Get<TestDbContext>();
        dbContext.Database.EnsureDeleted();
        dbContext.Dispose();
    }

    [BeforeScenario("@smoke")]
    public void BeforeSmokeTest()
    {
        // Configuración específica para pruebas smoke
        Console.WriteLine("Ejecutando prueba smoke crítica");
    }

    [AfterStep]
    public void AfterStep()
    {
        // Capturar screenshot si el paso falla
        if (_scenarioContext.TestError != null)
        {
            var driver = _scenarioContext.Get<IWebDriver>();
            var screenshot = ((ITakesScreenshot)driver).GetScreenshot();
            screenshot.SaveAsFile($"error_{DateTime.Now:yyyyMMddHHmmss}.png");
        }
    }
}
```

---

## Recomendaciones de Selección

### Para Pruebas Unitarias

**Elige xUnit si:**
- Trabajas en proyectos .NET Core/.NET 5+
- Valoras código limpio y mejores prácticas
- Tu equipo tiene experiencia en testing
- Trabajas en proyectos open source

**Elige MSTest si:**
- Trabajas en entornos enterprise tradicionales
- Necesitas soporte oficial de Microsoft
- Tu equipo es nuevo en testing
- Migras desde proyectos legacy

**Elige NUnit si:**
- Necesitas assertions muy expresivas
- Trabajas con pruebas parametrizadas complejas
- Ya tienes experiencia con NUnit
- Migras desde Java/JUnit

### Para Mocking

**Elige Moq si:**
- Es tu primer proyecto con mocking (es el estándar)
- Necesitas abundantes recursos y ejemplos
- Valoras una sintaxis expresiva
- Trabajas en un equipo grande

**Elige NSubstitute si:**
- Priorizas simplicidad sobre funcionalidades avanzadas
- Tu equipo es nuevo en mocking
- Valoras código limpio y minimalista
- No necesitas verificaciones complejas

**Elige FakeItEasy si:**
- Necesitas mensajes de error excepcionales
- Valoras consistencia en la API
- La terminología tradicional de mocking confunde a tu equipo
- El debugging de pruebas es frecuente

### Para Cobertura de Código

**Elige OpenCover si:**
- Trabajas con .NET Framework 2.0 a 4.8
- Necesitas una solución gratuita y probada
- Quieres integración con CI/CD
- No tienes presupuesto para herramientas comerciales
- Requieres compatibilidad con reportes estándar (Cobertura, OpenCover XML)

**Elige dotCover si:**
- Tienes licencia de JetBrains o presupuesto para adquirirla
- Valoras experiencia de usuario superior
- Necesitas análisis en tiempo real en el editor
- Trabajas principalmente en Windows con Visual Studio
- La productividad del equipo justifica el costo

**Elige Visual Studio Enterprise Code Coverage si:**
- Ya tienes licencias de Visual Studio Enterprise
- Valoras la integración nativa sin herramientas externas
- No quieres gestionar dependencias adicionales
- Trabajas en entorno puramente Microsoft

### Para Pruebas de UI

**Elige Playwright si:**
- Comienzas un proyecto nuevo
- Valoras velocidad y estabilidad
- Necesitas características modernas (interceptación de red)
- Tu aplicación es una SPA moderna

**Elige Selenium si:**
- Mantienes pruebas legacy existentes
- Necesitas soporte para navegadores exóticos
- Tu equipo ya conoce Selenium
- Requieres máxima compatibilidad

---

## Configuración de un Proyecto Completo

### Estructura de proyecto recomendada

```
MiProyecto/
├── src/
│   ├── MiProyecto.Core/              # Lógica de negocio
│   ├── MiProyecto.Infrastructure/    # Acceso a datos, servicios externos
│   └── MiProyecto.Web/               # API o Web App
├── tests/
│   ├── MiProyecto.UnitTests/         # Pruebas unitarias
│   ├── MiProyecto.IntegrationTests/  # Pruebas de integración
│   ├── MiProyecto.UITests/           # Pruebas de interfaz
│   └── MiProyecto.SpecFlow/          # Pruebas BDD
├── tools/
│   └── coverage-report/              # Reportes de cobertura
├── .editorconfig                     # Configuración de analyzers
├── stylecop.json                     # Configuración de StyleCop
└── sonar-project.properties          # Configuración de SonarQube
```

### Archivo packages.config para proyecto de pruebas .NET Framework

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <!-- Framework de pruebas -->
  <package id="MSTest.TestFramework" version="3.1.1" targetFramework="net48" />
  <package id="MSTest.TestAdapter" version="3.1.1" targetFramework="net48" />
  <!-- O alternativamente xUnit o NUnit -->
  <!-- <package id="xunit" version="2.6.2" targetFramework="net48" />
  <package id="xunit.runner.visualstudio" version="2.5.4" targetFramework="net48" /> -->
  
  <!-- Mocking -->
  <package id="Moq" version="4.20.70" targetFramework="net48" />
  
  <!-- Cobertura -->
  <package id="OpenCover" version="4.7.1221" targetFramework="net48" />
  <package id="ReportGenerator" version="5.1.26" targetFramework="net48" />
  
  <!-- Analyzers -->
  <package id="Microsoft.CodeAnalysis.NetAnalyzers" version="8.0.0" targetFramework="net48" 
           developmentDependency="true" />
  <package id="StyleCop.Analyzers" version="1.2.0-beta.507" targetFramework="net48" 
           developmentDependency="true" />
  
  <!-- Testing de Web API -->
  <package id="Microsoft.AspNet.WebApi.SelfHost" version="5.3.0" targetFramework="net48" />
  <package id="Microsoft.AspNet.WebApi.Client" version="5.3.0" targetFramework="net48" />
  
  <!-- Utilidades -->
  <package id="FluentAssertions" version="6.12.0" targetFramework="net48" />
  <package id="Bogus" version="35.3.0" targetFramework="net48" />
  <package id="EntityFramework" version="6.4.4" targetFramework="net48" />
  <package id="Newtonsoft.Json" version="13.0.3" targetFramework="net48" />
  
  <!-- Integration Testing -->
  <package id="Respawn" version="6.1.0" targetFramework="net48" />
</packages>
```

### Archivo .csproj tradicional para proyecto de pruebas

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.props" Condition="Exists('..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.props')" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{8E9F3C1A-1234-5678-9ABC-DEF012345678}</ProjectGuid>
    <OutputType>Library</OutputType>
    <AppDesignerFolder>Properties</AppDesignerFolder>
    <RootNamespace>MiProyecto.UnitTests</RootNamespace>
    <AssemblyName>MiProyecto.UnitTests</AssemblyName>
    <TargetFrameworkVersion>v4.8</TargetFrameworkVersion>
    <FileAlignment>512</FileAlignment>
    <ProjectTypeGuids>{3AC096D0-A1C2-E12C-1390-A8335801FDAB};{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}</ProjectTypeGuids>
    <VisualStudioVersion Condition="'$(VisualStudioVersion)' == ''">15.0</VisualStudioVersion>
    <VSToolsPath Condition="'$(VSToolsPath)' == ''">$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)</VSToolsPath>
    <ReferencePath>$(ProgramFiles)\Common Files\microsoft shared\VSTT\$(VisualStudioVersion)\UITestExtensionPackages</ReferencePath>
    <IsCodedUITest>False</IsCodedUITest>
    <TestProjectType>UnitTest</TestProjectType>
    <NuGetPackageImportStamp>
    </NuGetPackageImportStamp>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <DebugSymbols>true</DebugSymbols>
    <DebugType>full</DebugType>
    <Optimize>false</Optimize>
    <OutputPath>bin\Debug\</OutputPath>
    <DefineConstants>DEBUG;TRACE</DefineConstants>
    <ErrorReport>prompt</ErrorReport>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
    <DebugType>pdbonly</DebugType>
    <Optimize>true</Optimize>
    <OutputPath>bin\Release\</OutputPath>
    <DefineConstants>TRACE</DefineConstants>
    <ErrorReport>prompt</ErrorReport>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="Microsoft.VisualStudio.TestPlatform.TestFramework, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a, processorArchitecture=MSIL">
      <HintPath>..\packages\MSTest.TestFramework.3.1.1\lib\net462\Microsoft.VisualStudio.TestPlatform.TestFramework.dll</HintPath>
    </Reference>
    <Reference Include="Microsoft.VisualStudio.TestPlatform.TestFramework.Extensions, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a, processorArchitecture=MSIL">
      <HintPath>..\packages\MSTest.TestFramework.3.1.1\lib\net462\Microsoft.VisualStudio.TestPlatform.TestFramework.Extensions.dll</HintPath>
    </Reference>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="Moq, Version=4.20.70.0, Culture=neutral, PublicKeyToken=69f491c39445e920, processorArchitecture=MSIL">
      <HintPath>..\packages\Moq.4.20.70\lib\net462\Moq.dll</HintPath>
    </Reference>
  </ItemGroup>
  <ItemGroup>
    <Compile Include="CalculadoraTests.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
  </ItemGroup>
  <ItemGroup>
    <None Include="packages.config" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\MiProyecto.Core\MiProyecto.Core.csproj">
      <Project>{12345678-1234-1234-1234-123456789ABC}</Project>
      <Name>MiProyecto.Core</Name>
    </ProjectReference>
  </ItemGroup>
  <Import Project="$(VSToolsPath)\TeamTest\Microsoft.TestTools.targets" Condition="Exists('$(VSToolsPath)\TeamTest\Microsoft.TestTools.targets')" />
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
  <Target Name="EnsureNuGetPackageBuildImports" BeforeTargets="PrepareForBuild">
    <PropertyGroup>
      <ErrorText>Este proyecto hace referencia a los paquetes NuGet que faltan en este equipo. Use la restauración de paquetes NuGet para descargarlos. Para obtener más información, consulte http://go.microsoft.com/fwlink/?LinkID=322105. El archivo que falta es {0}.</ErrorText>
    </PropertyGroup>
    <Error Condition="!Exists('..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.props')" Text="$([System.String]::Format('$(ErrorText)', '..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.props'))" />
    <Error Condition="!Exists('..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.targets')" Text="$([System.String]::Format('$(ErrorText)', '..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.targets'))" />
  </Target>
  <Import Project="..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.targets" Condition="Exists('..\packages\MSTest.TestAdapter.3.1.1\build\net462\MSTest.TestAdapter.targets')" />
</Project>
```

### Script de CI/CD completo para .NET Framework

```yaml
# azure-pipelines.yml para .NET Framework
trigger:
  - main
  - develop

pool:
  vmImage: 'windows-latest'  # .NET Framework requiere Windows

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildAndTest
    steps:
    # Instalar NuGet Tool Installer
    - task: NuGetToolInstaller@1
      displayName: 'Install NuGet'

    # Restaurar paquetes NuGet
    - task: NuGetCommand@2
      displayName: 'Restore NuGet packages'
      inputs:
        restoreSolution: '$(solution)'

    # Compilar solución con MSBuild
    - task: VSBuild@1
      displayName: 'Build solution'
      inputs:
        solution: '$(solution)'
        msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.artifactStagingDirectory)"'
        platform: '$(buildPlatform)'
        configuration: '$(buildConfiguration)'

    # Ejecutar pruebas unitarias con VSTest
    - task: VSTest@2
      displayName: 'Run unit tests'
      inputs:
        platform: '$(buildPlatform)'
        configuration: '$(buildConfiguration)'
        codeCoverageEnabled: true

    # Generar cobertura con OpenCover
    - script: |
        nuget install OpenCover -Version 4.7.1221 -OutputDirectory $(Build.SourcesDirectory)\packages
        nuget install ReportGenerator -Version 5.1.26 -OutputDirectory $(Build.SourcesDirectory)\packages
        
        $(Build.SourcesDirectory)\packages\OpenCover.4.7.1221\tools\OpenCover.Console.exe ^
          -target:"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" ^
          -targetargs:"**\bin\$(buildConfiguration)\*Tests.dll" ^
          -filter:"+[MiProyecto*]* -[*.Tests]*" ^
          -register:user ^
          -output:coverage.xml
        
        $(Build.SourcesDirectory)\packages\ReportGenerator.5.1.26\tools\net47\ReportGenerator.exe ^
          -reports:coverage.xml ^
          -targetdir:$(Build.ArtifactStagingDirectory)\coveragereport ^
          -reporttypes:HtmlInline_AzurePipelines;Cobertura
      displayName: 'Generate code coverage with OpenCover'
      condition: succeededOrFailed()

    # Publicar resultados de cobertura
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Build.ArtifactStagingDirectory)\coveragereport\Cobertura.xml'
        reportDirectory: '$(Build.ArtifactStagingDirectory)\coveragereport'

    # Análisis con SonarQube (opcional)
    - task: SonarQubePrepare@5
      inputs:
        SonarQube: 'SonarQube Connection'
        scannerMode: 'MSBuild'
        projectKey: 'mi-proyecto'
        projectName: 'Mi Proyecto'

    - task: SonarQubeAnalyze@5

    - task: SonarQubePublish@5
      inputs:
        pollingTimeoutSec: '300'

    # Publicar artefactos
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'
```

### Script batch alternativo para ejecución local

```batch
@echo off
echo ========================================
echo Ejecutando Build y Tests con Cobertura
echo ========================================

REM Restaurar paquetes
echo Restaurando paquetes NuGet...
nuget restore MiProyecto.sln

REM Compilar solución
echo Compilando solución...
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe" ^
  MiProyecto.sln ^
  /p:Configuration=Release ^
  /p:Platform="Any CPU"

if %errorlevel% neq 0 (
    echo Error en la compilacion
    exit /b %errorlevel%
)

REM Ejecutar pruebas
echo Ejecutando pruebas unitarias...
vstest.console.exe ^
  .\MiProyecto.UnitTests\bin\Release\MiProyecto.UnitTests.dll ^
  /Logger:trx

REM Generar cobertura con OpenCover
echo Generando cobertura de código...
packages\OpenCover.4.7.1221\tools\OpenCover.Console.exe ^
  -target:"vstest.console.exe" ^
  -targetargs:".\MiProyecto.UnitTests\bin\Release\MiProyecto.UnitTests.dll" ^
  -filter:"+[MiProyecto*]* -[*.Tests]*" ^
  -register:user ^
  -output:coverage.xml

REM Generar reporte HTML
echo Generando reporte de cobertura...
packages\ReportGenerator.5.1.26\tools\net47\ReportGenerator.exe ^
  -reports:coverage.xml ^
  -targetdir:coveragereport ^
  -reporttypes:Html

echo.
echo ========================================
echo Build completado exitosamente
echo Reporte de cobertura: coveragereport\index.html
echo ========================================

start coveragereport\index.html
```
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/*coverage.cobertura.xml'

    - script: |
        dotnet tool install --global dotnet-reportgenerator-globaltool
        reportgenerator -reports:$(Agent.TempDirectory)/**/*coverage.cobertura.xml -targetdir:$(Build.ArtifactStagingDirectory)/coveragereport -reporttypes:Html
      displayName: 'Generate coverage report'

    - task: SonarCloudPrepare@1
      inputs:
        SonarCloud: 'SonarCloud Connection'
        organization: 'mi-organizacion'
        scannerMode: 'MSBuild'
        projectKey: 'mi-proyecto'

    - task: SonarCloudAnalyze@1

    - task: SonarCloudPublish@1
      inputs:
        pollingTimeoutSec: '300'

    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'coverage-report'
```

---

## Referencias y Recursos

### Documentación oficial

**Frameworks de pruebas:**
- xUnit: https://xunit.net/
- MSTest: https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest
- NUnit: https://nunit.org/

**Mocking:**
- Moq: https://github.com/moq/moq4
- NSubstitute: https://nsubstitute.github.io/
- FakeItEasy: https://fakeiteasy.github.io/

**Cobertura y análisis:**
- Coverlet: https://github.com/coverlet-coverage/coverlet
- SonarQube: https://www.sonarqube.org/
- Roslyn Analyzers: https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/overview

**Pruebas de UI:**
- Selenium: https://www.selenium.dev/documentation/
- Playwright: https://playwright.dev/dotnet/

**BDD:**
- SpecFlow: https://specflow.org/

### Libros recomendados

- "Unit Testing Principles, Practices, and Patterns" - Vladimir Khorikov
- "The Art of Unit Testing" - Roy Osherove
- "xUnit Test Patterns" - Gerard Meszaros
- "Test Driven Development: By Example" - Kent Beck

### Cursos y tutoriales

- Microsoft Learn: Testing in .NET
- Pluralsight: Testing .NET Applications
- LinkedIn Learning: Advanced Unit Testing
- YouTube: Nick Chapsas (canal especializado en .NET testing)

### Comunidades y blogs

- Stack Overflow: etiqueta [.net] [unit-testing]
- Reddit: r/dotnet, r/programming
- Blog de Vladimir Khorikov: https://enterprisecraftsmanship.com/
- Blog de Mark Seemann: https://blog.ploeh.dk/

---

## Conclusión

La elección de herramientas y frameworks de testing en .NET debe basarse en las necesidades específicas del proyecto, las habilidades del equipo y el contexto organizacional. No existe una solución única que sea la mejor para todos los casos.

**Principios generales a seguir:**

1. **Empieza simple:** Comienza con las herramientas más populares y probadas (xUnit + Moq + Coverlet) antes de explorar alternativas.

2. **Consistencia sobre preferencias:** Es más importante que todo el equipo use las mismas herramientas que cada desarrollador use su favorita.

3. **Evalúa continuamente:** Las herramientas evolucionan, lo que era mejor hace 5 años puede no serlo hoy.

4. **Prioriza lo que aporta valor:** Usa herramientas que realmente mejoren la calidad del código, no solo porque están de moda.

5. **Invierte en aprendizaje:** Las herramientas son solo el medio, el conocimiento de buenas prácticas de testing es el fin.

Con el conocimiento adquirido en este tema, estás preparado para tomar decisiones informadas sobre qué herramientas y frameworks usar en tus proyectos .NET, adaptándolas a las necesidades específicas de tu contexto.

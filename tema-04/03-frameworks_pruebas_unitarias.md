# Frameworks de Pruebas Unitarias: MSTest, xUnit, NUnit

## Comparativa de Frameworks

| Característica | MSTest | xUnit | NUnit |
|----------------|--------|-------|-------|
| **Desarrollador** | Microsoft | Microsoft/Comunidad | Comunidad |
| **Integración VS** | Nativa | Excelente | Excelente |
| **Paralelización** | Limitada | Por defecto | Configurable |
| **Setup/Teardown** | Métodos por clase | Constructor/Dispose | Atributos flexibles |
| **Extensibilidad** | Media | Alta | Alta |
| **Sintaxis** | Tradicional | Moderna | Tradicional |
| **Popularidad** | Alta en Enterprise | Creciente | Muy Alta |

## MSTest

Framework oficial de Microsoft, integrado nativamente en Visual Studio. Ideal para proyectos corporativos con fuerte dependencia del ecosistema Microsoft.

### Atributos Principales

**[TestClass]**
- Marca una clase como contenedora de pruebas
```csharp
[TestClass]
public class CalculadoraTests { }
```

**[TestMethod]**
- Define un método de prueba
```csharp
[TestMethod]
public void Sumar_DosNumeros_RetornaResultadoCorrecto() { }
```

**[TestInitialize]**
- Se ejecuta antes de cada prueba
```csharp
[TestInitialize]
public void Setup() { }
```

**[TestCleanup]**
- Se ejecuta después de cada prueba
```csharp
[TestCleanup]
public void Cleanup() { }
```

**[ClassInitialize]**
- Se ejecuta una vez antes de todas las pruebas de la clase
```csharp
[ClassInitialize]
public static void SetupClass(TestContext context) { }
```

**[ClassCleanup]**
- Se ejecuta una vez después de todas las pruebas
```csharp
[ClassCleanup]
public static void CleanupClass() { }
```

**[Ignore]**
- Omite temporalmente una prueba
```csharp
[Ignore("Pendiente de revisión")]
[TestMethod]
public void PruebaEnDesarrollo() { }
```

**[DataRow]** / **[DataTestMethod]**
- Pruebas parametrizadas
```csharp
[DataTestMethod]
[DataRow(2, 3, 5)]
[DataRow(5, 5, 10)]
public void Sumar_ValoresParametrizados(int a, int b, int esperado) { }
```

**[ExpectedException]**
- Verifica que se lance una excepción específica
```csharp
[ExpectedException(typeof(DivideByZeroException))]
[TestMethod]
public void Dividir_EntreCero_LanzaExcepcion() { }
```

## xUnit

Framework moderno y ligero, preferido por el equipo de ASP.NET Core. Promueve mejores prácticas con diseño minimalista.

### Atributos Principales

**[Fact]**
- Define una prueba sin parámetros
```csharp
[Fact]
public void Sumar_DosNumeros_RetornaResultadoCorrecto() { }
```

**[Theory]**
- Define una prueba parametrizada
```csharp
[Theory]
[InlineData(2, 3, 5)]
[InlineData(5, 5, 10)]
public void Sumar_ValoresParametrizados(int a, int b, int esperado) { }
```

**[InlineData]**
- Proporciona datos en línea para Theory
```csharp
[InlineData("test", 4)]
[InlineData("prueba", 6)]
```

**[MemberData]**
- Datos desde propiedades o métodos
```csharp
[MemberData(nameof(GetDatosPrueba))]
public static IEnumerable<object[]> GetDatosPrueba() { }
```

**[ClassData]**
- Datos desde una clase dedicada
```csharp
[ClassData(typeof(DatosCalculadora))]
```

**[Skip]**
- Omite una prueba con razón
```csharp
[Fact(Skip = "Funcionalidad no implementada")]
public void PruebaFutura() { }
```

**[Trait]**
- Categoriza pruebas para filtrado
```csharp
[Trait("Categoria", "Integracion")]
[Fact]
public void PruebaIntegracion() { }
```

### Características Especiales de xUnit

**Constructor y IDisposable** - Reemplaza Setup/Teardown
```csharp
public class CalculadoraTests : IDisposable
{
    private readonly Calculadora _calc;
    
    public CalculadoraTests() { _calc = new Calculadora(); }
    
    public void Dispose() { /* Limpieza */ }
}
```

**IClassFixture<T>** - Contexto compartido entre pruebas
```csharp
public class DatabaseTests : IClassFixture<DatabaseFixture> { }
```

## NUnit

Framework veterano con amplia adopción en la comunidad. Ofrece el conjunto más rico de atributos y funcionalidades.

### Atributos Principales

**[TestFixture]**
- Marca una clase de pruebas
```csharp
[TestFixture]
public class CalculadoraTests { }
```

**[Test]**
- Define un método de prueba
```csharp
[Test]
public void Sumar_DosNumeros_RetornaResultadoCorrecto() { }
```

**[SetUp]**
- Se ejecuta antes de cada prueba
```csharp
[SetUp]
public void Setup() { }
```

**[TearDown]**
- Se ejecuta después de cada prueba
```csharp
[TearDown]
public void Teardown() { }
```

**[OneTimeSetUp]**
- Se ejecuta una vez antes de todas las pruebas
```csharp
[OneTimeSetUp]
public void OneTimeSetup() { }
```

**[OneTimeTearDown]**
- Se ejecuta una vez después de todas las pruebas
```csharp
[OneTimeTearDown]
public void OneTimeTeardown() { }
```

**[TestCase]**
- Pruebas parametrizadas inline
```csharp
[TestCase(2, 3, 5)]
[TestCase(5, 5, 10)]
public void Sumar_ValoresParametrizados(int a, int b, int esperado) { }
```

**[TestCaseSource]**
- Datos desde método o propiedad
```csharp
[TestCaseSource(nameof(GetDatosPrueba))]
public void PruebaConDatosExternos(int a, int b) { }
```

**[Ignore]**
- Omite una prueba con razón
```csharp
[Ignore("Requiere refactorización")]
[Test]
public void PruebaPendiente() { }
```

**[Category]**
- Categoriza pruebas
```csharp
[Category("Integracion")]
[Test]
public void PruebaIntegracion() { }
```

**[Timeout]**
- Establece tiempo máximo de ejecución
```csharp
[Timeout(2000)]
[Test]
public void PruebaConLimite() { }
```

**[Repeat]**
- Ejecuta una prueba múltiples veces
```csharp
[Repeat(5)]
[Test]
public void PruebaRepetida() { }
```

**[Order]**
- Define orden de ejecución
```csharp
[Order(1)]
[Test]
public void PrimeraPrueba() { }
```

**[Values]** / **[Range]**
- Valores individuales o rangos
```csharp
[Test]
public void Prueba([Values(1, 2, 3)] int valor) { }

[Test]
public void Prueba([Range(1, 5)] int numero) { }
```

## Recomendaciones de Uso

**MSTest** - Cuando:
- Trabajas en entornos corporativos Microsoft
- Necesitas integración nativa con Visual Studio
- El equipo está familiarizado con herramientas Microsoft

**xUnit** - Cuando:
- Buscas sintaxis moderna y limpia
- Prefieres paralelización por defecto
- Desarrollas aplicaciones .NET modernas

**NUnit** - Cuando:
- Necesitas funcionalidad avanzada y flexible
- Requieres control detallado sobre ejecución
- Migras desde otros lenguajes (similar a JUnit)

## Ejemplo Comparativo

### MSTest
```csharp
[TestClass]
public class CalculadoraTests
{
    [TestMethod]
    [DataRow(2, 3, 5)]
    public void Sumar_Test(int a, int b, int esperado)
    {
        var calc = new Calculadora();
        Assert.AreEqual(esperado, calc.Sumar(a, b));
    }
}
```

### xUnit
```csharp
public class CalculadoraTests
{
    [Theory]
    [InlineData(2, 3, 5)]
    public void Sumar_Test(int a, int b, int esperado)
    {
        var calc = new Calculadora();
        Assert.Equal(esperado, calc.Sumar(a, b));
    }
}
```

### NUnit
```csharp
[TestFixture]
public class CalculadoraTests
{
    [TestCase(2, 3, 5)]
    public void Sumar_Test(int a, int b, int esperado)
    {
        var calc = new Calculadora();
        Assert.AreEqual(esperado, calc.Sumar(a, b));
    }
}
```

## Conclusión

Los tres frameworks son sólidos y capaces. La elección depende del contexto del proyecto, preferencias del equipo y necesidades específicas. Para .NET 4.8, cualquiera de los tres ofrece soporte completo y estable.

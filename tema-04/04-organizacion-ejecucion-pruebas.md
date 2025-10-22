# Organización de pruebas en proyectos .NET

## Estructura de proyectos de pruebas

En .NET 4.8, la organización de pruebas sigue una estructura clara que facilita el mantenimiento:

### Convención de nombres
- **Proyecto de pruebas**: `NombreProyecto.Tests` o `NombreProyecto.UnitTests`
- **Clases de prueba**: `ClaseProbadaTests` o `ClaseProbadaTest`
- **Métodos de prueba**: `MetodoProbado_Escenario_ResultadoEsperado`

**Ejemplo:**
```
SistemaVentas/
├── SistemaVentas/
│   └── Servicios/
│       └── CalculadoraDescuentos.cs
└── SistemaVentas.Tests/
    └── Servicios/
        └── CalculadoraDescuentosTests.cs
```

### Organización de clases de prueba

```csharp
[TestClass]
public class CalculadoraDescuentosTests
{
    private CalculadoraDescuentos _calculadora;

    [TestInitialize]
    public void Setup()
    {
        _calculadora = new CalculadoraDescuentos();
    }

    [TestMethod]
    public void CalcularDescuento_ClienteVIP_RetornaDescuento20Porciento()
    {
        // Arrange
        var cliente = new Cliente { Tipo = TipoCliente.VIP };
        decimal precioOriginal = 100m;

        // Act
        var resultado = _calculadora.CalcularDescuento(cliente, precioOriginal);

        // Assert
        Assert.AreEqual(20m, resultado);
    }

    [TestCleanup]
    public void Cleanup()
    {
        _calculadora = null;
    }
}
```

### Carpetas por categoría

Organiza las pruebas en carpetas según su tipo:

```
SistemaVentas.Tests/
├── Unit/              # Pruebas unitarias
├── Integration/       # Pruebas de integración
└── TestHelpers/       # Utilidades compartidas
```

## Ejecución y depuración de pruebas

### Ejecución desde Visual Studio

**Ejecutar todas las pruebas:**
- Menú: `Test > Run All Tests`
- Atajo: `Ctrl + R, A`

**Ejecutar prueba individual:**
- Clic derecho en el método > `Run Test(s)`
- Atajo: `Ctrl + R, T` (con el cursor en el método)

**Test Explorer:**
- Ventana: `Test > Windows > Test Explorer`
- Permite filtrar, agrupar y ejecutar pruebas selectivamente

### Depuración de pruebas

**Iniciar depuración:**
- Clic derecho en la prueba > `Debug Test(s)`
- Atajo: `Ctrl + R, Ctrl + T`

**Establecer breakpoints:**
```csharp
[TestMethod]
public void ProcesarPedido_StockInsuficiente_LanzaExcepcion()
{
    // Arrange
    var producto = new Producto { Stock = 5 };
    var pedido = new Pedido { Cantidad = 10 };
    
    // Act & Assert - Breakpoint aquí para inspeccionar
    Assert.ThrowsException<StockInsuficienteException>(() => 
    {
        _servicio.ProcesarPedido(pedido, producto);
    });
}
```

### Ejecución desde línea de comandos

**MSTest con VSTest.Console:**
```bash
vstest.console.exe SistemaVentas.Tests.dll
```

**Con filtros:**
```bash
vstest.console.exe SistemaVentas.Tests.dll /TestCaseFilter:"TestCategory=Unit"
```

### Categorización de pruebas

```csharp
[TestClass]
public class ServicioPedidosTests
{
    [TestMethod]
    [TestCategory("Unit")]
    public void CalcularTotal_ConIVA_RetornaTotalCorrecto()
    {
        // Prueba unitaria rápida
    }

    [TestMethod]
    [TestCategory("Integration")]
    public void GuardarPedido_EnBaseDatos_SeGuardaCorrectamente()
    {
        // Prueba de integración más lenta
    }
}
```

### Análisis de resultados

**Test Explorer muestra:**
- ✓ Pruebas exitosas (verde)
- ✗ Pruebas fallidas (rojo)
- ⊘ Pruebas omitidas (amarillo)
- Tiempo de ejecución
- Mensajes de error y stack traces

### Mejores prácticas

1. **Ejecuta pruebas frecuentemente** durante el desarrollo
2. **Usa categorías** para separar pruebas rápidas de lentas
3. **Revisa el Test Explorer** antes de cada commit
4. **Depura pruebas fallidas** inmediatamente
5. **Mantén las pruebas rápidas** para ejecución constante

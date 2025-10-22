# Pruebas Unitarias en .NET 4.8 - Conceptos Clave

## Test Units (Unidades de Prueba)

Una **unidad de prueba** es la porción más pequeña de código que puede ser probada de forma aislada. En .NET 4.8, generalmente corresponde a:

- Un método individual
- Una clase con responsabilidad única
- Un componente que realiza una operación específica

**Características de una buena unidad de prueba:**
- **Independiente**: No depende de otras pruebas
- **Repetible**: Produce el mismo resultado siempre
- **Rápida**: Se ejecuta en milisegundos
- **Verificable**: Tiene un resultado claro (pasa o falla)

```csharp
// Ejemplo de unidad testeable
public class CalculadoraDescuento
{
    public decimal CalcularDescuento(decimal precio, decimal porcentaje)
    {
        if (precio < 0 || porcentaje < 0 || porcentaje > 100)
            throw new ArgumentException("Valores inválidos");
        
        return precio * (porcentaje / 100);
    }
}
```

---

## TDD (Test-Driven Development)

### ¿Qué es TDD?

**TDD** es una disciplina de desarrollo donde **primero se escribe la prueba** que falla, luego el código mínimo para que pase, y finalmente se refactoriza.

### El Ciclo Red-Green-Refactor

```
🔴 RED → 🟢 GREEN → 🔵 REFACTOR
   ↑                      ↓
   └──────────────────────┘
```

1. **🔴 RED (Rojo)**: Escribir una prueba que falle
2. **🟢 GREEN (Verde)**: Escribir el código mínimo para que pase
3. **🔵 REFACTOR (Refactorizar)**: Mejorar el código sin cambiar funcionalidad

### Ejemplo Completo de TDD

#### Paso 1: RED - Escribir la prueba que falla

```csharp
[TestClass]
public class ValidadorEmailTests
{
    [TestMethod]
    public void ValidarEmail_EmailValido_RetornaTrue()
    {
        // Arrange
        var validador = new ValidadorEmail();
        
        // Act
        bool resultado = validador.EsValido("usuario@dominio.com");
        
        // Assert
        Assert.IsTrue(resultado);
    }
}
// ❌ Esta prueba falla porque ValidadorEmail no existe
```

#### Paso 2: GREEN - Código mínimo que pasa

```csharp
public class ValidadorEmail
{
    public bool EsValido(string email)
    {
        return true; // Implementación mínima
    }
}
// ✅ La prueba pasa
```

#### Paso 3: RED - Nueva prueba que falle

```csharp
[TestMethod]
public void ValidarEmail_EmailSinArroba_RetornaFalse()
{
    // Arrange
    var validador = new ValidadorEmail();
    
    // Act
    bool resultado = validador.EsValido("usuariodominio.com");
    
    // Assert
    Assert.IsFalse(resultado);
}
// ❌ Falla porque siempre retorna true
```

#### Paso 4: GREEN - Mejorar implementación

```csharp
public class ValidadorEmail
{
    public bool EsValido(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            return false;
            
        return email.Contains("@");
    }
}
// ✅ Ambas pruebas pasan
```

#### Paso 5: REFACTOR - Mejorar el código

```csharp
public class ValidadorEmail
{
    private static readonly Regex RegexEmail = 
        new Regex(@"^[^@\s]+@[^@\s]+\.[^@\s]+$", RegexOptions.Compiled);
    
    public bool EsValido(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            return false;
            
        return RegexEmail.IsMatch(email);
    }
}
// ✅ Todas las pruebas siguen pasando con mejor implementación
```

### Beneficios de TDD

- **Diseño emergente**: El código se diseña para ser testeable
- **Documentación viva**: Las pruebas documentan el comportamiento esperado
- **Confianza en refactorización**: Puedes mejorar código sin romper funcionalidad
- **Menos defectos**: Los bugs se detectan tempranamente
- **Cobertura natural**: El código tiene pruebas por diseño

### Cuándo usar TDD

✅ **Recomendado para:**
- Lógica de negocio compleja
- Algoritmos y cálculos
- Validaciones y reglas de negocio
- Código que cambia frecuentemente

❌ **No siempre necesario en:**
- Prototipos rápidos
- Código de configuración simple
- UI/Presentación básica

---

## AAA (Arrange-Act-Assert)

### Patrón de Estructura

**AAA** es el patrón estándar para estructurar pruebas unitarias de forma clara y legible.

```csharp
[TestMethod]
public void NombrePrueba_Escenario_ResultadoEsperado()
{
    // Arrange (Preparar)
    // Configurar datos y dependencias
    
    // Act (Actuar)
    // Ejecutar el método a probar
    
    // Assert (Afirmar)
    // Verificar el resultado
}
```

### Desglose de cada fase

#### 1. **Arrange (Preparar)**

Configura el contexto necesario para la prueba:
- Crear instancias de objetos
- Inicializar variables
- Configurar mocks y stubs
- Establecer estado inicial

```csharp
// Arrange
var repositorio = new Mock<IClienteRepositorio>();
var servicio = new ClienteServicio(repositorio.Object);
var clienteId = 123;
repositorio.Setup(r => r.ObtenerPorId(clienteId))
    .Returns(new Cliente { Id = clienteId, Nombre = "Juan" });
```

#### 2. **Act (Actuar)**

Ejecuta la operación que se está probando:
- Llamar al método bajo prueba
- Debe ser **una sola acción**
- Capturar el resultado o excepción

```csharp
// Act
var resultado = servicio.ObtenerCliente(clienteId);
```

#### 3. **Assert (Afirmar)**

Verifica que el resultado es el esperado:
- Comprobar valores de retorno
- Verificar estado de objetos
- Validar que se llamaron métodos
- Confirmar excepciones lanzadas

```csharp
// Assert
Assert.IsNotNull(resultado);
Assert.AreEqual("Juan", resultado.Nombre);
Assert.AreEqual(clienteId, resultado.Id);
```

### Ejemplo Completo con AAA

```csharp
[TestClass]
public class CarritoComprasTests
{
    [TestMethod]
    public void AgregarProducto_ProductoValido_IncrementaCantidad()
    {
        // Arrange
        var carrito = new CarritoCompras();
        var producto = new Producto 
        { 
            Id = 1, 
            Nombre = "Laptop", 
            Precio = 999.99m 
        };
        
        // Act
        carrito.AgregarProducto(producto, cantidad: 2);
        
        // Assert
        Assert.AreEqual(1, carrito.TotalProductos);
        Assert.AreEqual(2, carrito.ObtenerCantidad(producto.Id));
        Assert.AreEqual(1999.98m, carrito.Total);
    }
    
    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void AgregarProducto_CantidadNegativa_LanzaExcepcion()
    {
        // Arrange
        var carrito = new CarritoCompras();
        var producto = new Producto { Id = 1, Nombre = "Laptop", Precio = 999.99m };
        
        // Act
        carrito.AgregarProducto(producto, cantidad: -1);
        
        // Assert - El atributo ExpectedException valida la excepción
    }
}
```

### Variaciones del patrón

**Given-When-Then** (BDD):
```csharp
[TestMethod]
public void DeberiaCalcularDescuentoPorVolumen()
{
    // Given (Dado que)
    var calculadora = new CalculadoraPrecios();
    var cantidad = 100;
    
    // When (Cuando)
    var precioFinal = calculadora.CalcularConDescuento(cantidad);
    
    // Then (Entonces)
    Assert.AreEqual(850m, precioFinal);
}
```

---

## ATDD (Acceptance Test-Driven Development)

### ¿Qué es ATDD?

**ATDD** es una evolución de TDD que se enfoca en escribir pruebas de aceptación antes del desarrollo, basándose en criterios de negocio y colaboración con stakeholders.

### Diferencias entre TDD y ATDD

| Aspecto | TDD | ATDD |
|---------|-----|------|
| **Enfoque** | Técnico | Negocio |
| **Nivel** | Unitario | Aceptación |
| **Participantes** | Desarrolladores | Equipo completo + Cliente |
| **Lenguaje** | Código | Lenguaje natural |
| **Objetivo** | Código correcto | Funcionalidad correcta |

### Proceso ATDD

1. **Discusión**: Equipo y cliente definen criterios de aceptación
2. **Documentación**: Escribir pruebas en lenguaje de negocio
3. **Desarrollo**: Implementar usando TDD
4. **Demostración**: Ejecutar pruebas de aceptación con el cliente

### Ejemplo de Criterios de Aceptación

**Historia de Usuario:**
> Como cliente, quiero aplicar un cupón de descuento para reducir el precio de mi compra

**Criterios de Aceptación:**

```gherkin
Escenario: Aplicar cupón válido del 20%
  Dado que tengo un carrito con total de 100€
  Y tengo un cupón "VERANO20" válido del 20%
  Cuando aplico el cupón al carrito
  Entonces el total debe ser 80€
  Y el cupón debe marcarse como usado

Escenario: Aplicar cupón expirado
  Dado que tengo un carrito con total de 100€
  Y tengo un cupón "ANTIGUO" que expiró el 01/01/2024
  Cuando intento aplicar el cupón
  Entonces debe rechazarse con mensaje "Cupón expirado"
  Y el total debe permanecer en 100€
```

### Implementación en .NET 4.8

```csharp
[TestClass]
public class AplicarCuponFeatureTests
{
    private CarritoCompras _carrito;
    private ServicioCupones _servicioCupones;
    
    [TestInitialize]
    public void Setup()
    {
        _carrito = new CarritoCompras();
        _servicioCupones = new ServicioCupones();
    }
    
    [TestMethod]
    public void AplicarCupon_CuponValido20Porciento_ReduceTotal()
    {
        // Dado que tengo un carrito con total de 100€
        _carrito.AgregarProducto(new Producto { Precio = 100m });
        
        // Y tengo un cupón "VERANO20" válido del 20%
        var cupon = new Cupon 
        { 
            Codigo = "VERANO20", 
            Descuento = 20, 
            FechaExpiracion = DateTime.Now.AddDays(30) 
        };
        
        // Cuando aplico el cupón al carrito
        var resultado = _servicioCupones.AplicarCupon(_carrito, cupon);
        
        // Entonces el total debe ser 80€
        Assert.IsTrue(resultado.Exitoso);
        Assert.AreEqual(80m, _carrito.Total);
        
        // Y el cupón debe marcarse como usado
        Assert.IsTrue(cupon.Usado);
    }
    
    [TestMethod]
    public void AplicarCupon_CuponExpirado_Rechaza()
    {
        // Dado que tengo un carrito con total de 100€
        _carrito.AgregarProducto(new Producto { Precio = 100m });
        
        // Y tengo un cupón que expiró
        var cupon = new Cupon 
        { 
            Codigo = "ANTIGUO", 
            Descuento = 20, 
            FechaExpiracion = new DateTime(2024, 1, 1) 
        };
        
        // Cuando intento aplicar el cupón
        var resultado = _servicioCupones.AplicarCupon(_carrito, cupon);
        
        // Entonces debe rechazarse con mensaje
        Assert.IsFalse(resultado.Exitoso);
        Assert.AreEqual("Cupón expirado", resultado.Mensaje);
        
        // Y el total debe permanecer en 100€
        Assert.AreEqual(100m, _carrito.Total);
    }
}
```

### Beneficios de ATDD

- **Entendimiento compartido**: Todo el equipo comprende los requisitos
- **Requisitos claros**: Criterios de aceptación específicos y verificables
- **Menos retrabajo**: Se detectan malentendidos antes de codificar
- **Documentación ejecutable**: Las pruebas son especificaciones vivas
- **Colaboración**: Fomenta comunicación entre negocio y técnicos

---

## Resumen de Conceptos

| Concepto | Propósito | Nivel |
|----------|-----------|-------|
| **Test Unit** | Probar la unidad más pequeña de código | Método/Clase |
| **TDD** | Diseñar código mediante pruebas primero | Desarrollo |
| **AAA** | Estructurar pruebas de forma clara | Organización |
| **ATDD** | Validar requisitos de negocio | Aceptación |

### Flujo Completo

```
ATDD (Requisitos) 
    ↓
TDD (Implementación)
    ↓
AAA (Estructura)
    ↓
Test Units (Ejecución)
```

---

## Mejores Prácticas

1. **Nombrado descriptivo**: `MetodoAPorbar_Escenario_ResultadoEsperado`
2. **Una aserción por concepto**: Enfocarse en validar un comportamiento
3. **Independencia**: Cada prueba debe ejecutarse sola
4. **Datos representativos**: Usar valores realistas
5. **Mantenibilidad**: Código de prueba tan limpio como el de producción
6. **Velocidad**: Las pruebas unitarias deben ser rápidas (<100ms)

---

## Ejemplo Integrado Final

```csharp
// Siguiendo TDD con estructura AAA para implementar ATDD
[TestClass]
public class ProcesadorPedidosTests
{
    // Criterio ATDD: Un pedido con stock suficiente debe procesarse correctamente
    [TestMethod]
    public void ProcesarPedido_StockSuficiente_PedidoConfirmado()
    {
        // Arrange - Preparar el escenario
        var mockInventario = new Mock<IInventario>();
        mockInventario.Setup(i => i.ConsultarStock(1)).Returns(50);
        
        var procesador = new ProcesadorPedidos(mockInventario.Object);
        var pedido = new Pedido 
        { 
            ProductoId = 1, 
            Cantidad = 10 
        };
        
        // Act - Ejecutar la operación
        var resultado = procesador.Procesar(pedido);
        
        // Assert - Verificar el resultado
        Assert.IsTrue(resultado.Exitoso);
        Assert.AreEqual(EstadoPedido.Confirmado, pedido.Estado);
        mockInventario.Verify(i => i.ReducirStock(1, 10), Times.Once);
    }
}
```

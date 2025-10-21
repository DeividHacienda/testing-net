# XML Documentation Comments en .NET

## Introducción

Los **XML Documentation Comments** son comentarios especiales en C# que permiten generar documentación automática del código. Se escriben usando una sintaxis XML específica y comienzan con tres barras diagonales `///`.

Estos comentarios sirven para:
- Documentar el propósito y comportamiento de clases, métodos, propiedades, etc.
- Generar documentación HTML/PDF automáticamente
- Proporcionar IntelliSense en Visual Studio y otros IDEs
- Facilitar el mantenimiento del código

---

## 1. La etiqueta `<summary>`

La etiqueta `<summary>` proporciona una descripción breve del elemento documentado. Es la etiqueta más importante y común.

### Ejemplo básico de clase:

```csharp
/// <summary>
/// Representa un cliente del sistema de facturación.
/// Contiene información personal y de contacto del cliente.
/// </summary>
public class Cliente
{
    public string Nombre { get; set; }
    public string Email { get; set; }
}
```

### Ejemplo de método:

```csharp
/// <summary>
/// Calcula el precio total de un producto aplicando el descuento especificado.
/// </summary>
public decimal CalcularPrecioConDescuento(decimal precioBase, decimal porcentajeDescuento)
{
    return precioBase - (precioBase * porcentajeDescuento / 100);
}
```

### Ejemplo de propiedad:

```csharp
/// <summary>
/// Obtiene o establece el número de identificación fiscal del cliente.
/// Debe ser un NIF válido español (8 dígitos + letra).
/// </summary>
public string NIF { get; set; }
```

### ✅ Buenas prácticas para `<summary>`:
- Escribir frases completas que terminen en punto
- Ser conciso pero descriptivo (1-3 líneas)
- Usar tercera persona ("Calcula..." no "Calculo...")
- Explicar QUÉ hace, no CÓMO lo hace

---

## 2. La etiqueta `<param>`

La etiqueta `<param>` documenta cada parámetro de un método. Debe haber una etiqueta por cada parámetro.

### Sintaxis:
```xml
/// <param name="nombreParametro">Descripción del parámetro</param>
```

### Ejemplo completo:

```csharp
/// <summary>
/// Registra un nuevo usuario en el sistema con validación de credenciales.
/// </summary>
/// <param name="nombreUsuario">Nombre de usuario único. Debe tener entre 3 y 20 caracteres alfanuméricos.</param>
/// <param name="email">Dirección de correo electrónico del usuario. Debe ser un email válido.</param>
/// <param name="contrasena">Contraseña del usuario. Debe tener al menos 8 caracteres, incluir mayúsculas, minúsculas y números.</param>
/// <param name="fechaNacimiento">Fecha de nacimiento del usuario. El usuario debe ser mayor de 18 años.</param>
public bool RegistrarUsuario(
    string nombreUsuario, 
    string email, 
    string contrasena, 
    DateTime fechaNacimiento)
{
    // Implementación...
    return true;
}
```

### Ejemplo con tipos complejos:

```csharp
/// <summary>
/// Procesa un pedido completo incluyendo validación de stock y cálculo de envío.
/// </summary>
/// <param name="pedido">Objeto pedido que contiene todos los artículos, dirección de envío y datos de pago.</param>
/// <param name="metodoPago">Método de pago seleccionado (tarjeta, transferencia, efectivo).</param>
/// <param name="aplicarDescuentoFidelizacion">Indica si se debe aplicar el descuento por fidelización al cliente.</param>
public ResultadoPedido ProcesarPedido(
    Pedido pedido, 
    MetodoPago metodoPago, 
    bool aplicarDescuentoFidelizacion)
{
    // Implementación...
    return new ResultadoPedido();
}
```

### ✅ Buenas prácticas para `<param>`:
- Describir el propósito del parámetro, no solo repetir su tipo
- Incluir restricciones o valores válidos
- Mencionar valores por defecto si los hay
- Usar frases cortas (no necesitan terminar en punto)

---

## 3. La etiqueta `<returns>`

La etiqueta `<returns>` documenta el valor de retorno de un método o propiedad.

### Ejemplo con tipo primitivo:

```csharp
/// <summary>
/// Calcula el índice de masa corporal (IMC) de una persona.
/// </summary>
/// <param name="peso">Peso de la persona en kilogramos.</param>
/// <param name="altura">Altura de la persona en metros.</param>
/// <returns>
/// El índice de masa corporal calculado como peso/altura². 
/// Valores típicos: menor a 18.5 (bajo peso), 18.5-24.9 (normal), 25-29.9 (sobrepeso), 30+ (obesidad).
/// </returns>
public double CalcularIMC(double peso, double altura)
{
    return peso / (altura * altura);
}
```

### Ejemplo con tipo complejo:

```csharp
/// <summary>
/// Busca productos en el catálogo según criterios específicos.
/// </summary>
/// <param name="categoria">Categoría de productos a buscar.</param>
/// <param name="precioMinimo">Precio mínimo del rango de búsqueda.</param>
/// <param name="precioMaximo">Precio máximo del rango de búsqueda.</param>
/// <returns>
/// Una lista de objetos Producto que coinciden con los criterios de búsqueda.
/// Retorna una lista vacía si no se encuentran coincidencias.
/// La lista está ordenada por relevancia de forma descendente.
/// </returns>
public List<Producto> BuscarProductos(
    string categoria, 
    decimal precioMinimo, 
    decimal precioMaximo)
{
    // Implementación...
    return new List<Producto>();
}
```

### Ejemplo con valores null:

```csharp
/// <summary>
/// Obtiene el último pedido realizado por un cliente específico.
/// </summary>
/// <param name="clienteId">Identificador único del cliente.</param>
/// <returns>
/// El objeto Pedido más reciente del cliente.
/// Retorna null si el cliente no tiene pedidos o si el clienteId no existe.
/// </returns>
public Pedido ObtenerUltimoPedido(int clienteId)
{
    // Implementación...
    return null;
}
```

### Ejemplo con tipo booleano:

```csharp
/// <summary>
/// Valida si una dirección de correo electrónico tiene un formato correcto.
/// </summary>
/// <param name="email">Dirección de correo electrónico a validar.</param>
/// <returns>
/// true si el email tiene un formato válido según RFC 5322; 
/// false en caso contrario.
/// </returns>
public bool EsEmailValido(string email)
{
    // Implementación...
    return true;
}
```

### ✅ Buenas prácticas para `<returns>`:
- Describir el significado del valor retornado, no solo su tipo
- Especificar valores especiales (null, -1, string.Empty, etc.)
- Explicar el estado del objeto retornado
- Para colecciones, indicar si pueden estar vacías
- Para booleanos, explicar qué significa true y false

---

## 4. La etiqueta `<exception>`

La etiqueta `<exception>` documenta las excepciones que un método puede lanzar. Debe haber una etiqueta por cada tipo de excepción.

### Sintaxis:
```xml
/// <exception cref="TipoExcepcion">Descripción de cuándo se lanza</exception>
```

### Ejemplo básico:

```csharp
/// <summary>
/// Divide dos números enteros.
/// </summary>
/// <param name="dividendo">Número a dividir.</param>
/// <param name="divisor">Número por el cual dividir.</param>
/// <returns>El resultado de la división.</returns>
/// <exception cref="DivideByZeroException">
/// Se lanza cuando el divisor es cero.
/// </exception>
public int Dividir(int dividendo, int divisor)
{
    if (divisor == 0)
        throw new DivideByZeroException("No se puede dividir entre cero");
    
    return dividendo / divisor;
}
```

### Ejemplo con múltiples excepciones:

```csharp
/// <summary>
/// Crea una nueva cuenta bancaria para un cliente.
/// </summary>
/// <param name="clienteId">Identificador del cliente propietario de la cuenta.</param>
/// <param name="saldoInicial">Saldo inicial de la cuenta.</param>
/// <param name="tipoCuenta">Tipo de cuenta a crear (ahorro, corriente, etc.).</param>
/// <returns>El objeto CuentaBancaria recién creado con su número de cuenta asignado.</returns>
/// <exception cref="ArgumentNullException">
/// Se lanza cuando clienteId es null o vacío.
/// </exception>
/// <exception cref="ArgumentException">
/// Se lanza cuando saldoInicial es negativo o cuando tipoCuenta no es un valor válido.
/// </exception>
/// <exception cref="ClienteNoEncontradoException">
/// Se lanza cuando no existe un cliente con el clienteId especificado.
/// </exception>
/// <exception cref="LimiteClientesExcedidoException">
/// Se lanza cuando el cliente ya tiene el número máximo de cuentas permitidas (5).
/// </exception>
public CuentaBancaria CrearCuenta(
    string clienteId, 
    decimal saldoInicial, 
    TipoCuenta tipoCuenta)
{
    if (string.IsNullOrEmpty(clienteId))
        throw new ArgumentNullException(nameof(clienteId));
    
    if (saldoInicial < 0)
        throw new ArgumentException("El saldo inicial no puede ser negativo", nameof(saldoInicial));
    
    // Más validaciones e implementación...
    return new CuentaBancaria();
}
```

### Ejemplo con excepciones de operaciones asíncronas:

```csharp
/// <summary>
/// Descarga un archivo desde una URL y lo guarda en el disco local.
/// </summary>
/// <param name="url">URL completa del archivo a descargar.</param>
/// <param name="rutaDestino">Ruta local donde se guardará el archivo.</param>
/// <returns>Una tarea que representa la operación asíncrona.</returns>
/// <exception cref="ArgumentNullException">
/// Se lanza cuando url o rutaDestino son null o vacíos.
/// </exception>
/// <exception cref="HttpRequestException">
/// Se lanza cuando hay un error de red o el servidor responde con un código de error HTTP.
/// </exception>
/// <exception cref="UnauthorizedAccessException">
/// Se lanza cuando no hay permisos para escribir en la rutaDestino.
/// </exception>
/// <exception cref="IOException">
/// Se lanza cuando hay un error al escribir el archivo en disco.
/// </exception>
/// <exception cref="TaskCanceledException">
/// Se lanza cuando la operación excede el tiempo de espera configurado (30 segundos).
/// </exception>
public async Task DescargarArchivoAsync(string url, string rutaDestino)
{
    if (string.IsNullOrEmpty(url))
        throw new ArgumentNullException(nameof(url));
    
    // Implementación...
}
```

### ✅ Buenas prácticas para `<exception>`:
- Documentar TODAS las excepciones que el método puede lanzar directamente
- Describir claramente las condiciones que causan la excepción
- No documentar excepciones que lance código interno que no se propagan
- Usar `cref` correctamente para referenciar el tipo de excepción
- Incluir excepciones de operaciones asíncronas comunes

---

## 5. La etiqueta `<remarks>`

La etiqueta `<remarks>` proporciona información adicional, detalles de implementación, notas importantes o ejemplos de uso que no caben en el `<summary>`.

### Ejemplo con notas de implementación:

```csharp
/// <summary>
/// Encripta una cadena de texto usando el algoritmo AES-256.
/// </summary>
/// <param name="textoPlano">Texto a encriptar.</param>
/// <param name="clave">Clave de encriptación de 32 bytes.</param>
/// <returns>El texto encriptado en formato Base64.</returns>
/// <remarks>
/// Este método utiliza AES-256 en modo CBC con padding PKCS7.
/// El vector de inicialización (IV) se genera aleatoriamente y se incluye
/// en los primeros 16 bytes del resultado.
/// 
/// <para>
/// <strong>Importante:</strong> La clave debe mantenerse segura y nunca 
/// debe almacenarse en texto plano en el código fuente.
/// </para>
/// 
/// <para>
/// Este método NO es adecuado para encriptar grandes volúmenes de datos.
/// Para archivos grandes, considere usar encriptación por streaming.
/// </para>
/// </remarks>
public string Encriptar(string textoPlano, byte[] clave)
{
    // Implementación...
    return string.Empty;
}
```

### Ejemplo con notas de rendimiento:

```csharp
/// <summary>
/// Ordena una lista de elementos usando el algoritmo QuickSort.
/// </summary>
/// <typeparam name="T">Tipo de elementos a ordenar. Debe implementar IComparable.</typeparam>
/// <param name="lista">Lista de elementos a ordenar.</param>
/// <remarks>
/// <para>
/// <strong>Complejidad temporal:</strong>
/// - Mejor caso: O(n log n)
/// - Caso promedio: O(n log n)
/// - Peor caso: O(n²)
/// </para>
/// 
/// <para>
/// <strong>Complejidad espacial:</strong> O(log n) debido a la recursión.
/// </para>
/// 
/// <para>
/// Este método modifica la lista original. Si necesita preservar la lista
/// original, haga una copia antes de llamar a este método.
/// </para>
/// 
/// <para>
/// Para listas con menos de 10 elementos, se usa Insertion Sort en su lugar
/// por motivos de eficiencia.
/// </para>
/// </remarks>
public void OrdenarQuickSort<T>(List<T> lista) where T : IComparable<T>
{
    // Implementación...
}
```

### Ejemplo con ejemplos de uso:

```csharp
/// <summary>
/// Formatea un número de teléfono español a un formato estándar.
/// </summary>
/// <param name="telefono">Número de teléfono sin formato.</param>
/// <returns>Número de teléfono formateado como "+34 XXX XX XX XX".</returns>
/// <exception cref="ArgumentException">
/// Se lanza cuando el teléfono no tiene un formato válido.
/// </exception>
/// <remarks>
/// Este método acepta múltiples formatos de entrada y los normaliza
/// al formato internacional español.
/// 
/// <para>
/// <strong>Ejemplos de uso:</strong>
/// </para>
/// 
/// <code>
/// // Desde un número sin formato
/// string resultado1 = FormatearTelefonoEspanol("666123456");
/// // Resultado: "+34 666 12 34 56"
/// 
/// // Desde un número con espacios
/// string resultado2 = FormatearTelefonoEspanol("666 12 34 56");
/// // Resultado: "+34 666 12 34 56"
/// 
/// // Desde un número con prefijo
/// string resultado3 = FormatearTelefonoEspanol("+34666123456");
/// // Resultado: "+34 666 12 34 56"
/// </code>
/// 
/// <para>
/// <strong>Nota:</strong> Este método solo funciona con números españoles.
/// Para otros países, use FormatearTelefonoInternacional().
/// </para>
/// </remarks>
public string FormatearTelefonoEspanol(string telefono)
{
    // Implementación...
    return string.Empty;
}
```

### Ejemplo con información de hilo (thread-safety):

```csharp
/// <summary>
/// Caché en memoria para almacenar resultados de consultas frecuentes.
/// </summary>
/// <remarks>
/// <para>
/// <strong>Thread Safety:</strong> Esta clase es thread-safe para operaciones
/// de lectura concurrentes. Las operaciones de escritura están protegidas
/// internamente con locks.
/// </para>
/// 
/// <para>
/// <strong>Política de expiración:</strong>
/// - Los elementos expiran automáticamente después de 5 minutos de inactividad
/// - El tamaño máximo de la caché es 1000 elementos
/// - Cuando se alcanza el límite, se eliminan los elementos menos usados (LRU)
/// </para>
/// 
/// <para>
/// <strong>Consideraciones de memoria:</strong>
/// Esta caché mantiene referencias fuertes a los objetos almacenados.
/// Para objetos grandes, considere usar WeakReference o limitar el tamaño de la caché.
/// </para>
/// 
/// <para>
/// <strong>Ejemplo de uso:</strong>
/// </para>
/// <code>
/// var cache = new CacheMemoria&lt;string, Usuario&gt;();
/// cache.Agregar("user123", usuario);
/// Usuario usuarioRecuperado = cache.Obtener("user123");
/// </code>
/// </remarks>
public class CacheMemoria<TKey, TValue>
{
    // Implementación...
}
```

### Ejemplo con historial de versiones:

```csharp
/// <summary>
/// Procesa pagos con tarjeta de crédito a través de la pasarela de pago.
/// </summary>
/// <param name="datosTarjeta">Información de la tarjeta de crédito.</param>
/// <param name="monto">Monto a cobrar en euros.</param>
/// <returns>El identificador de la transacción si el pago fue exitoso.</returns>
/// <remarks>
/// <para>
/// <strong>Seguridad:</strong> Este método cumple con PCI-DSS nivel 1.
/// Los datos de la tarjeta no se almacenan en ningún momento.
/// Toda la comunicación con la pasarela usa TLS 1.3.
/// </para>
/// 
/// <para>
/// <strong>Límites:</strong>
/// - Monto mínimo: 0.50 €
/// - Monto máximo: 5,000.00 €
/// - Reintentos automáticos: 3 veces con backoff exponencial
/// </para>
/// 
/// <para>
/// <strong>Historial de cambios:</strong>
/// - v2.0: Añadido soporte para 3D Secure 2.0
/// - v1.5: Mejoras en manejo de errores y reintentos
/// - v1.0: Versión inicial con soporte para Visa y Mastercard
/// </para>
/// </remarks>
public async Task<string> ProcesarPagoAsync(
    DatosTarjeta datosTarjeta, 
    decimal monto)
{
    // Implementación...
    return "TXN123456";
}
```

### ✅ Buenas prácticas para `<remarks>`:
- Usar cuando `<summary>` no es suficiente
- Incluir información técnica importante
- Documentar comportamientos no obvios
- Añadir ejemplos de código con la etiqueta `<code>`
- Usar `<para>` para separar párrafos
- Documentar consideraciones de rendimiento, seguridad o thread-safety
- Mencionar alternativas o métodos relacionados

---

## 6. Ejemplo completo de clase documentada

Aquí un ejemplo completo que integra todas las etiquetas:

```csharp
/// <summary>
/// Gestiona las operaciones de un carrito de compras online.
/// Permite agregar, eliminar y modificar productos, así como calcular totales.
/// </summary>
/// <remarks>
/// <para>
/// Esta clase mantiene el estado del carrito en memoria durante la sesión del usuario.
/// Para persistencia entre sesiones, use CarritoRepository para guardar y recuperar el estado.
/// </para>
/// 
/// <para>
/// <strong>Thread Safety:</strong> Esta clase NO es thread-safe. 
/// Cada usuario debe tener su propia instancia del carrito.
/// </para>
/// </remarks>
public class CarritoCompras
{
    /// <summary>
    /// Obtiene la lista de artículos actualmente en el carrito.
    /// </summary>
    /// <remarks>
    /// Esta propiedad retorna una copia de solo lectura de la lista interna
    /// para prevenir modificaciones externas no controladas.
    /// </remarks>
    public IReadOnlyList<ArticuloCarrito> Articulos { get; private set; }

    /// <summary>
    /// Obtiene el importe total del carrito sin aplicar descuentos ni impuestos.
    /// </summary>
    /// <returns>
    /// El subtotal calculado sumando el precio unitario por cantidad de cada artículo.
    /// </returns>
    public decimal Subtotal => Articulos.Sum(a => a.PrecioUnitario * a.Cantidad);

    /// <summary>
    /// Agrega un producto al carrito de compras.
    /// Si el producto ya existe, incrementa su cantidad.
    /// </summary>
    /// <param name="productoId">Identificador único del producto a agregar.</param>
    /// <param name="cantidad">Cantidad de unidades a agregar. Debe ser mayor a cero.</param>
    /// <param name="precioUnitario">Precio por unidad del producto en euros.</param>
    /// <returns>
    /// true si el producto se agregó exitosamente; 
    /// false si ya existía y solo se actualizó la cantidad.
    /// </returns>
    /// <exception cref="ArgumentException">
    /// Se lanza cuando cantidad es menor o igual a cero, o cuando precioUnitario es negativo.
    /// </exception>
    /// <exception cref="ArgumentNullException">
    /// Se lanza cuando productoId es null o vacío.
    /// </exception>
    /// <exception cref="StockInsuficienteException">
    /// Se lanza cuando no hay suficiente stock disponible para la cantidad solicitada.
    /// </exception>
    /// <remarks>
    /// <para>
    /// Este método verifica automáticamente el stock disponible antes de agregar el producto.
    /// Si el stock es insuficiente, se lanza una excepción y el carrito no se modifica.
    /// </para>
    /// 
    /// <para>
    /// <strong>Ejemplo de uso:</strong>
    /// </para>
    /// <code>
    /// var carrito = new CarritoCompras();
    /// bool agregado = carrito.AgregarProducto("PROD123", 2, 19.99m);
    /// 
    /// if (agregado)
    /// {
    ///     Console.WriteLine("Producto agregado al carrito");
    /// }
    /// else
    /// {
    ///     Console.WriteLine("Cantidad actualizada para producto existente");
    /// }
    /// </code>
    /// </remarks>
    public bool AgregarProducto(string productoId, int cantidad, decimal precioUnitario)
    {
        if (string.IsNullOrEmpty(productoId))
            throw new ArgumentNullException(nameof(productoId), 
                "El ID del producto no puede ser null o vacío");

        if (cantidad <= 0)
            throw new ArgumentException(
                "La cantidad debe ser mayor a cero", nameof(cantidad));

        if (precioUnitario < 0)
            throw new ArgumentException(
                "El precio unitario no puede ser negativo", nameof(precioUnitario));

        // Verificar stock
        var stockDisponible = ObtenerStockDisponible(productoId);
        if (stockDisponible < cantidad)
            throw new StockInsuficienteException(
                $"Stock insuficiente. Disponible: {stockDisponible}, Solicitado: {cantidad}");

        // Implementación...
        return true;
    }

    /// <summary>
    /// Elimina un producto del carrito de compras.
    /// </summary>
    /// <param name="productoId">Identificador único del producto a eliminar.</param>
    /// <returns>
    /// true si el producto fue eliminado exitosamente; 
    /// false si el producto no estaba en el carrito.
    /// </returns>
    /// <exception cref="ArgumentNullException">
    /// Se lanza cuando productoId es null o vacío.
    /// </exception>
    /// <remarks>
    /// Este método no lanza excepción si el producto no existe en el carrito,
    /// simplemente retorna false. Esto facilita el manejo de la UI.
    /// </remarks>
    public bool EliminarProducto(string productoId)
    {
        if (string.IsNullOrEmpty(productoId))
            throw new ArgumentNullException(nameof(productoId));

        // Implementación...
        return true;
    }

    /// <summary>
    /// Calcula el total del carrito aplicando descuentos e impuestos.
    /// </summary>
    /// <param name="codigoDescuento">Código de descuento promocional opcional. Puede ser null.</param>
    /// <param name="tasaImpuesto">Tasa de impuesto a aplicar en porcentaje (ej: 21 para 21% de IVA).</param>
    /// <returns>
    /// Un objeto ResumenCarrito que contiene el subtotal, descuentos, impuestos y total final.
    /// </returns>
    /// <exception cref="ArgumentException">
    /// Se lanza cuando tasaImpuesto es negativa o mayor a 100.
    /// </exception>
    /// <exception cref="CodigoDescuentoInvalidoException">
    /// Se lanza cuando el código de descuento no es válido o ha expirado.
    /// </exception>
    /// <remarks>
    /// <para>
    /// El cálculo se realiza en el siguiente orden:
    /// 1. Se calcula el subtotal sumando todos los productos
    /// 2. Se aplica el descuento si hay un código válido
    /// 3. Se calcula el impuesto sobre el subtotal con descuento
    /// 4. Se suma todo para obtener el total final
    /// </para>
    /// 
    /// <para>
    /// Los descuentos pueden ser porcentuales o de importe fijo, dependiendo
    /// del tipo de código promocional aplicado.
    /// </para>
    /// 
    /// <para>
    /// <strong>Ejemplo de uso:</strong>
    /// </para>
    /// <code>
    /// var carrito = new CarritoCompras();
    /// carrito.AgregarProducto("PROD123", 2, 19.99m);
    /// carrito.AgregarProducto("PROD456", 1, 49.99m);
    /// 
    /// // Calcular con descuento e IVA del 21%
    /// var resumen = carrito.CalcularTotal("VERANO2024", 21);
    /// 
    /// Console.WriteLine($"Subtotal: {resumen.Subtotal}€");
    /// Console.WriteLine($"Descuento: {resumen.Descuento}€");
    /// Console.WriteLine($"IVA (21%): {resumen.Impuesto}€");
    /// Console.WriteLine($"Total: {resumen.Total}€");
    /// </code>
    /// </remarks>
    public ResumenCarrito CalcularTotal(string codigoDescuento, decimal tasaImpuesto)
    {
        if (tasaImpuesto < 0 || tasaImpuesto > 100)
            throw new ArgumentException(
                "La tasa de impuesto debe estar entre 0 y 100", nameof(tasaImpuesto));

        // Implementación...
        return new ResumenCarrito();
    }

    /// <summary>
    /// Vacía completamente el carrito eliminando todos los productos.
    /// </summary>
    /// <remarks>
    /// Esta operación no se puede deshacer. 
    /// Use con precaución especialmente en flujos de checkout.
    /// </remarks>
    public void VaciarCarrito()
    {
        // Implementación...
    }

    // Método auxiliar privado (no se documenta públicamente)
    private int ObtenerStockDisponible(string productoId)
    {
        // Implementación...
        return 100;
    }
}

/// <summary>
/// Contiene el resumen financiero del carrito de compras.
/// </summary>
public class ResumenCarrito
{
    /// <summary>
    /// Obtiene el subtotal antes de descuentos e impuestos.
    /// </summary>
    public decimal Subtotal { get; set; }

    /// <summary>
    /// Obtiene el importe total de descuentos aplicados.
    /// </summary>
    public decimal Descuento { get; set; }

    /// <summary>
    /// Obtiene el importe total de impuestos.
    /// </summary>
    public decimal Impuesto { get; set; }

    /// <summary>
    /// Obtiene el importe total final a pagar.
    /// </summary>
    public decimal Total { get; set; }
}

/// <summary>
/// Excepción que se lanza cuando se intenta agregar más cantidad de un producto
/// de la que hay disponible en stock.
/// </summary>
public class StockInsuficienteException : Exception
{
    /// <summary>
    /// Inicializa una nueva instancia de StockInsuficienteException.
    /// </summary>
    /// <param name="mensaje">Mensaje descriptivo del error.</param>
    public StockInsuficienteException(string mensaje) : base(mensaje) { }
}

/// <summary>
/// Excepción que se lanza cuando se intenta aplicar un código de descuento inválido o expirado.
/// </summary>
public class CodigoDescuentoInvalidoException : Exception
{
    /// <summary>
    /// Inicializa una nueva instancia de CodigoDescuentoInvalidoException.
    /// </summary>
    /// <param name="mensaje">Mensaje descriptivo del error.</param>
    public CodigoDescuentoInvalidoException(string mensaje) : base(mensaje) { }
}
```

---

## 7. Etiquetas adicionales útiles

### `<example>` - Ejemplos de uso

```csharp
/// <summary>
/// Convierte una temperatura de Celsius a Fahrenheit.
/// </summary>
/// <param name="celsius">Temperatura en grados Celsius.</param>
/// <returns>Temperatura convertida a grados Fahrenheit.</returns>
/// <example>
/// Este ejemplo muestra cómo usar el método ConvertiCelsiusToFahrenheit:
/// <code>
/// var convertidor = new ConvertidorTemperatura();
/// double fahrenheit = convertidor.ConvertirCelsiusToFahrenheit(25);
/// Console.WriteLine($"25°C son {fahrenheit}°F"); // Imprime: 25°C son 77°F
/// </code>
/// </example>
public double ConvertirCelsiusToFahrenheit(double celsius)
{
    return (celsius * 9 / 5) + 32;
}
```

### `<see>` y `<seealso>` - Referencias cruzadas

```csharp
/// <summary>
/// Valida un número de tarjeta de crédito usando el algoritmo de Luhn.
/// </summary>
/// <param name="numeroTarjeta">Número de tarjeta sin espacios ni guiones.</param>
/// <returns>true si la tarjeta es válida; false en caso contrario.</returns>
/// <remarks>
/// Para validar otros aspectos de la tarjeta, vea también 
/// <see cref="ValidarFechaExpiracion"/> y <see cref="ValidarCVV"/>.
/// </remarks>
/// <seealso cref="ValidarFechaExpiracion"/>
/// <seealso cref="ValidarCVV"/>
public bool ValidarNumeroTarjeta(string numeroTarjeta)
{
    // Implementación...
    return true;
}
```

### `<typeparam>` - Parámetros de tipo genérico

```csharp
/// <summary>
/// Repositorio genérico para operaciones CRUD en base de datos.
/// </summary>
/// <typeparam name="T">Tipo de entidad que maneja el repositorio. Debe ser una clase con ID.</typeparam>
/// <remarks>
/// Este repositorio proporciona operaciones básicas de Create, Read, Update y Delete
/// para cualquier entidad que implemente IEntidad.
/// </remarks>
public class RepositorioGenerico<T> where T : class, IEntidad
{
    /// <summary>
    /// Obtiene una entidad por su identificador.
    /// </summary>
    /// <param name="id">Identificador único de la entidad.</param>
    /// <returns>La entidad encontrada o null si no existe.</returns>
    public T ObtenerPorId(int id)
    {
        // Implementación...
        return null;
    }
}
```

### `<value>` - Descripción de propiedades

```csharp
/// <summary>
/// Representa una configuración de aplicación.
/// </summary>
public class ConfiguracionApp
{
    /// <summary>
    /// Obtiene o establece la cadena de conexión a la base de datos.
    /// </summary>
    /// <value>
    /// Una cadena de conexión válida de SQL Server. 
    /// No debe contener credenciales en texto plano en producción.
    /// Ejemplo: "Server=myserver;Database=mydb;Integrated Security=true;"
    /// </value>
    public string CadenaConexion { get; set; }

    /// <summary>
    /// Obtiene o establece el nivel de logging de la aplicación.
    /// </summary>
    /// <value>
    /// Un valor de la enumeración NivelLog (Debug, Info, Warning, Error, Critical).
    /// Por defecto es Info.
    /// </value>
    public NivelLog NivelLogging { get; set; } = NivelLog.Info;
}
```

### `<code>` - Bloques de código de ejemplo

```csharp
/// <summary>
/// Cliente HTTP con reintentos automáticos y circuit breaker.
/// </summary>
/// <remarks>
/// <para>
/// Este cliente implementa un patrón de resiliencia con reintentos exponenciales
/// y circuit breaker para prevenir sobrecarga de servicios fallidos.
/// </para>
/// 
/// <example>
/// Ejemplo básico de uso:
/// <code>
/// var cliente = new HttpClientResilient();
/// 
/// // Configurar política de reintentos
/// cliente.ConfigurarReintentos(
///     intentosMaximos: 3,
///     tiempoEsperaInicial: TimeSpan.FromSeconds(1)
/// );
/// 
/// // Realizar petición con reintentos automáticos
/// var respuesta = await cliente.GetAsync("https://api.ejemplo.com/datos");
/// 
/// if (respuesta.IsSuccessStatusCode)
/// {
///     var contenido = await respuesta.Content.ReadAsStringAsync();
///     Console.WriteLine(contenido);
/// }
/// </code>
/// </example>
/// </remarks>
public class HttpClientResilient
{
    // Implementación...
}
```

---

## 8. Generación automática de documentación

Una vez documentado el código, puedes generar documentación HTML usando herramientas como:

### Configuración en el .csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <!-- Generar archivo XML de documentación -->
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <!-- Ruta personalizada (opcional) -->
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\MiProyecto.xml</DocumentationFile>
    <!-- No mostrar warnings por métodos sin documentar -->
    <NoWarn>$(NoWarn);1591</NoWarn>
  </PropertyGroup>
</Project>
```

### Herramientas para generar documentación

1. **DocFX**: Herramienta oficial de Microsoft
   ```bash
   dotnet tool install -g docfx
   docfx init
   docfx build
   ```

2. **Sandcastle Help File Builder**: Genera archivos de ayuda estilo MSDN

3. **Doxygen**: Soporta múltiples lenguajes incluyendo C#

---

## 9. Buenas prácticas generales

### ✅ QUÉ HACER:

1. **Documentar interfaces públicas**: Toda clase, método, propiedad pública debe estar documentada
2. **Ser descriptivo pero conciso**: Explicar QUÉ hace el código, no CÓMO
3. **Actualizar la documentación**: Cuando cambia el código, actualizar los comentarios
4. **Usar lenguaje claro**: Evitar jerga innecesaria
5. **Incluir ejemplos**: Para APIs complejas, incluir ejemplos de uso
6. **Documentar casos especiales**: Valores null, listas vacías, excepciones
7. **Usar tercera persona**: "Calcula el total" no "Calculo el total"

### ❌ QUÉ EVITAR:

1. **Comentarios obvios**: 
   ```csharp
   /// <summary>
   /// Obtiene o establece el nombre.
   /// </summary>
   public string Nombre { get; set; } // ❌ Obvio y sin valor
   ```

2. **Documentar implementación privada**: 
   - Métodos y clases privadas no necesitan XML documentation
   - Usar comentarios regulares `//` para detalles de implementación

3. **Copiar y pegar documentación**: 
   - Cada método debe tener su propia descripción específica

4. **Dejar documentación desactualizada**:
   - Peor que no tener documentación es tener documentación incorrecta

5. **Usar frases incompletas en `<summary>`**:
   ```csharp
   /// <summary>
   /// para calcular el precio // ❌ Frase incompleta
   /// </summary>
   ```

6. **No documentar excepciones**:
   - Siempre documentar qué excepciones puede lanzar un método

---

## 10. Checklist de documentación

Antes de dar por terminada la documentación de un método, verifica:

- [ ] ¿Tiene `<summary>` descriptivo?
- [ ] ¿Todos los `<param>` están documentados?
- [ ] ¿Está documentado `<returns>` (si aplica)?
- [ ] ¿Están documentadas todas las `<exception>` posibles?
- [ ] ¿Se necesita `<remarks>` para información adicional?
- [ ] ¿Los ejemplos de código funcionan correctamente?
- [ ] ¿La documentación es precisa con la implementación actual?
- [ ] ¿Se usó un lenguaje claro y profesional?
- [ ] ¿Se evitaron comentarios obvios o redundantes?

---

## Conclusión

Los XML Documentation Comments son una herramienta fundamental para:
- Crear APIs autodocumentadas
- Mejorar el IntelliSense y la experiencia del desarrollador
- Generar documentación profesional automáticamente
- Facilitar el mantenimiento del código
- Reducir la curva de aprendizaje para nuevos desarrolladores

**La documentación es código**: debe mantenerse, actualizarse y tratarse con el mismo cuidado que el código funcional.

---

## Referencias

- [Documentación oficial de Microsoft sobre XML Documentation](https://learn.microsoft.com/es-es/dotnet/csharp/language-reference/xmldoc/)
- [DocFX Documentation](https://dotnet.github.io/docfx/)
- [C# Coding Conventions](https://learn.microsoft.com/es-es/dotnet/csharp/fundamentals/coding-style/coding-conventions)

---

**Autor**: Material para el curso "Testing .Net y QA (avanzado)"  
**Fecha**: Octubre 2025  
**Versión**: 1.0

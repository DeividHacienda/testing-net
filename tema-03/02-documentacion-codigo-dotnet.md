# Documentación de Clases, Métodos, Propiedades e Interfaces en .NET

## Introducción

La documentación de código en .NET utiliza comentarios XML especiales que permiten generar documentación automática y proporcionar información contextual en tiempo de desarrollo a través de IntelliSense. Una documentación clara y completa mejora la mantenibilidad del código y facilita la colaboración en equipos.

## 1. Documentación de Clases

### 1.1 Sintaxis Básica

```csharp
/// <summary>
/// Representa un cliente en el sistema de gestión comercial.
/// </summary>
/// <remarks>
/// Esta clase maneja la información básica del cliente y sus operaciones relacionadas.
/// Implementa validaciones de datos según las reglas de negocio establecidas.
/// </remarks>
public class Cliente
{
    // Implementación
}
```

### 1.2 Elementos Recomendados para Clases

- **`<summary>`**: Descripción concisa del propósito de la clase
- **`<remarks>`**: Información adicional, contexto de uso, consideraciones especiales
- **`<example>`**: Ejemplos de uso cuando sea apropiado
- **`<seealso>`**: Referencias a clases relacionadas

### 1.3 Ejemplo Completo de Clase Documentada

```csharp
/// <summary>
/// Gestiona las operaciones de validación de tarjetas de crédito.
/// </summary>
/// <remarks>
/// Esta clase implementa el algoritmo de Luhn para validar números de tarjetas
/// y proporciona métodos adicionales para verificar tipos de tarjeta.
/// <para>
/// Soporta los siguientes tipos de tarjetas: Visa, MasterCard, American Express.
/// </para>
/// </remarks>
/// <example>
/// <code>
/// var validador = new ValidadorTarjeta();
/// bool esValida = validador.ValidarNumero("4532015112830366");
/// </code>
/// </example>
/// <seealso cref="TarjetaCredito"/>
/// <seealso cref="ProcesadorPagos"/>
public class ValidadorTarjeta
{
    // Implementación
}
```

### 1.4 Clases Genéricas

```csharp
/// <summary>
/// Repositorio genérico para operaciones CRUD sobre entidades.
/// </summary>
/// <typeparam name="T">El tipo de entidad que maneja el repositorio. Debe implementar IEntity.</typeparam>
/// <remarks>
/// Esta clase proporciona implementación base para operaciones comunes de persistencia.
/// </remarks>
public class RepositorioBase<T> where T : IEntity
{
    // Implementación
}
```

## 2. Documentación de Métodos

### 2.1 Elementos Esenciales

Los métodos deben documentar claramente su propósito, parámetros, valor de retorno y excepciones que pueden lanzar.

```csharp
/// <summary>
/// Calcula el total de una factura aplicando descuentos e impuestos.
/// </summary>
/// <param name="subtotal">El subtotal antes de aplicar descuentos e impuestos.</param>
/// <param name="porcentajeDescuento">Porcentaje de descuento a aplicar (0-100).</param>
/// <param name="aplicarIVA">Indica si se debe aplicar el IVA al cálculo.</param>
/// <returns>
/// El total final de la factura con descuentos e impuestos aplicados.
/// </returns>
/// <exception cref="ArgumentOutOfRangeException">
/// Se lanza cuando <paramref name="porcentajeDescuento"/> está fuera del rango 0-100.
/// </exception>
/// <exception cref="ArgumentException">
/// Se lanza cuando <paramref name="subtotal"/> es negativo.
/// </exception>
public decimal CalcularTotal(decimal subtotal, decimal porcentajeDescuento, bool aplicarIVA)
{
    if (subtotal < 0)
        throw new ArgumentException("El subtotal no puede ser negativo.", nameof(subtotal));
    
    if (porcentajeDescuento < 0 || porcentajeDescuento > 100)
        throw new ArgumentOutOfRangeException(nameof(porcentajeDescuento), 
            "El porcentaje debe estar entre 0 y 100.");
    
    decimal totalConDescuento = subtotal * (1 - porcentajeDescuento / 100);
    return aplicarIVA ? totalConDescuento * 1.21m : totalConDescuento;
}
```

### 2.2 Métodos Asíncronos

```csharp
/// <summary>
/// Obtiene los datos del cliente de forma asíncrona desde la base de datos.
/// </summary>
/// <param name="clienteId">Identificador único del cliente.</param>
/// <param name="cancellationToken">Token para cancelar la operación asíncrona.</param>
/// <returns>
/// Una tarea que representa la operación asíncrona. El resultado contiene los datos
/// del cliente o null si no se encuentra.
/// </returns>
/// <exception cref="DatabaseException">
/// Se lanza cuando ocurre un error al acceder a la base de datos.
/// </exception>
public async Task<Cliente> ObtenerClienteAsync(int clienteId, CancellationToken cancellationToken = default)
{
    // Implementación
}
```

### 2.3 Métodos con Parámetros Opcionales

```csharp
/// <summary>
/// Busca productos en el catálogo según los criterios especificados.
/// </summary>
/// <param name="nombre">Nombre o parte del nombre del producto a buscar.</param>
/// <param name="categoria">Categoría del producto. Si es null, busca en todas las categorías.</param>
/// <param name="precioMinimo">Precio mínimo del producto. Por defecto es 0.</param>
/// <param name="precioMaximo">Precio máximo del producto. Si es null, no aplica límite superior.</param>
/// <returns>Lista de productos que cumplen con los criterios de búsqueda.</returns>
public List<Producto> BuscarProductos(
    string nombre, 
    string categoria = null, 
    decimal precioMinimo = 0, 
    decimal? precioMaximo = null)
{
    // Implementación
}
```

### 2.4 Métodos Genéricos

```csharp
/// <summary>
/// Convierte una colección de objetos a una lista del tipo especificado.
/// </summary>
/// <typeparam name="TSource">Tipo de los elementos en la colección de origen.</typeparam>
/// <typeparam name="TDestination">Tipo de los elementos en la lista de destino.</typeparam>
/// <param name="source">Colección de origen a convertir.</param>
/// <param name="converter">Función que convierte cada elemento de TSource a TDestination.</param>
/// <returns>Lista de elementos convertidos al tipo TDestination.</returns>
/// <exception cref="ArgumentNullException">
/// Se lanza cuando <paramref name="source"/> o <paramref name="converter"/> son null.
/// </exception>
public List<TDestination> ConvertirLista<TSource, TDestination>(
    IEnumerable<TSource> source, 
    Func<TSource, TDestination> converter)
{
    // Implementación
}
```

## 3. Documentación de Propiedades

### 3.1 Propiedades Básicas

```csharp
/// <summary>
/// Obtiene o establece el nombre completo del cliente.
/// </summary>
/// <value>
/// Cadena que representa el nombre completo. No puede ser null o vacío.
/// </value>
/// <exception cref="ArgumentException">
/// Se lanza al intentar establecer un valor null o vacío.
/// </exception>
public string NombreCompleto 
{ 
    get => _nombreCompleto;
    set
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("El nombre no puede estar vacío.");
        _nombreCompleto = value;
    }
}
```

### 3.2 Propiedades de Solo Lectura

```csharp
/// <summary>
/// Obtiene el identificador único del cliente.
/// </summary>
/// <value>
/// Entero que representa el ID único asignado al cliente en el sistema.
/// Este valor se asigna automáticamente al crear el cliente y no puede modificarse.
/// </value>
public int Id { get; }
```

### 3.3 Propiedades Calculadas

```csharp
/// <summary>
/// Obtiene el nombre completo del cliente formateado.
/// </summary>
/// <value>
/// Cadena con formato "Apellido, Nombre" en mayúsculas.
/// </value>
/// <remarks>
/// Esta propiedad no almacena valor, se calcula dinámicamente cada vez que se accede.
/// </remarks>
public string NombreFormateado => $"{Apellido.ToUpper()}, {Nombre.ToUpper()}";
```

### 3.4 Propiedades con Validación Compleja

```csharp
/// <summary>
/// Obtiene o establece el correo electrónico del cliente.
/// </summary>
/// <value>
/// Cadena que debe contener una dirección de correo electrónico válida.
/// </value>
/// <exception cref="ArgumentException">
/// Se lanza cuando el valor no cumple con el formato de email válido.
/// </exception>
/// <example>
/// <code>
/// cliente.Email = "usuario@ejemplo.com"; // Válido
/// cliente.Email = "correo-invalido";     // Lanza ArgumentException
/// </code>
/// </example>
public string Email 
{ 
    get => _email;
    set
    {
        if (!EsEmailValido(value))
            throw new ArgumentException("El formato de email no es válido.");
        _email = value;
    }
}
```

## 4. Documentación de Interfaces

### 4.1 Interface Básica

```csharp
/// <summary>
/// Define el contrato para servicios de notificación.
/// </summary>
/// <remarks>
/// Las implementaciones de esta interfaz deben proporcionar funcionalidad
/// para enviar notificaciones a través de diferentes canales.
/// </remarks>
public interface IServicioNotificacion
{
    /// <summary>
    /// Envía una notificación al destinatario especificado.
    /// </summary>
    /// <param name="destinatario">Dirección o identificador del destinatario.</param>
    /// <param name="mensaje">Contenido del mensaje a enviar.</param>
    /// <returns>
    /// True si la notificación se envió correctamente; de lo contrario, false.
    /// </returns>
    bool EnviarNotificacion(string destinatario, string mensaje);
}
```

### 4.2 Interface Genérica

```csharp
/// <summary>
/// Define operaciones CRUD básicas para repositorios de datos.
/// </summary>
/// <typeparam name="T">Tipo de entidad que maneja el repositorio.</typeparam>
/// <typeparam name="TKey">Tipo de la clave primaria de la entidad.</typeparam>
/// <remarks>
/// Esta interfaz establece el contrato estándar para todos los repositorios
/// del sistema, garantizando consistencia en las operaciones de datos.
/// </remarks>
public interface IRepositorio<T, TKey> where T : class
{
    /// <summary>
    /// Obtiene una entidad por su identificador.
    /// </summary>
    /// <param name="id">Identificador de la entidad.</param>
    /// <returns>La entidad encontrada o null si no existe.</returns>
    Task<T> ObtenerPorIdAsync(TKey id);
    
    /// <summary>
    /// Obtiene todas las entidades del repositorio.
    /// </summary>
    /// <returns>Colección de todas las entidades.</returns>
    Task<IEnumerable<T>> ObtenerTodosAsync();
    
    /// <summary>
    /// Agrega una nueva entidad al repositorio.
    /// </summary>
    /// <param name="entidad">Entidad a agregar.</param>
    /// <returns>La entidad agregada con su ID asignado.</returns>
    /// <exception cref="ArgumentNullException">
    /// Se lanza cuando <paramref name="entidad"/> es null.
    /// </exception>
    Task<T> AgregarAsync(T entidad);
    
    /// <summary>
    /// Actualiza una entidad existente.
    /// </summary>
    /// <param name="entidad">Entidad con los datos actualizados.</param>
    /// <returns>True si se actualizó correctamente; false en caso contrario.</returns>
    Task<bool> ActualizarAsync(T entidad);
    
    /// <summary>
    /// Elimina una entidad por su identificador.
    /// </summary>
    /// <param name="id">Identificador de la entidad a eliminar.</param>
    /// <returns>True si se eliminó correctamente; false en caso contrario.</returns>
    Task<bool> EliminarAsync(TKey id);
}
```

### 4.3 Interface con Eventos

```csharp
/// <summary>
/// Define el contrato para procesadores de pedidos con notificación de eventos.
/// </summary>
public interface IProcesadorPedidos
{
    /// <summary>
    /// Se dispara cuando un pedido ha sido procesado exitosamente.
    /// </summary>
    event EventHandler<PedidoProcesadoEventArgs> PedidoProcesado;
    
    /// <summary>
    /// Se dispara cuando ocurre un error al procesar un pedido.
    /// </summary>
    event EventHandler<ErrorPedidoEventArgs> ErrorEnProcesamiento;
    
    /// <summary>
    /// Procesa un pedido de forma asíncrona.
    /// </summary>
    /// <param name="pedido">El pedido a procesar.</param>
    /// <returns>Resultado del procesamiento del pedido.</returns>
    Task<ResultadoProcesamiento> ProcesarPedidoAsync(Pedido pedido);
}
```

## 5. Etiquetas XML Especiales

### 5.1 `<para>` - Párrafos en Remarks

```csharp
/// <summary>
/// Gestiona la configuración de la aplicación.
/// </summary>
/// <remarks>
/// Esta clase proporciona acceso centralizado a la configuración.
/// <para>
/// La configuración se carga desde múltiples fuentes en el siguiente orden:
/// 1. Archivo appsettings.json
/// 2. Variables de entorno
/// 3. Azure Key Vault (en producción)
/// </para>
/// <para>
/// Los cambios en la configuración se detectan automáticamente y se recargan
/// sin necesidad de reiniciar la aplicación.
/// </para>
/// </remarks>
public class ConfiguracionManager
{
    // Implementación
}
```

### 5.2 `<code>` - Ejemplos de Código

```csharp
/// <summary>
/// Valida si una cadena cumple con el formato de un NIF español.
/// </summary>
/// <param name="nif">Cadena a validar.</param>
/// <returns>True si el formato es válido; false en caso contrario.</returns>
/// <example>
/// Ejemplos de uso:
/// <code>
/// var validador = new ValidadorDocumentos();
/// 
/// // Caso válido
/// bool esValido = validador.ValidarNIF("12345678Z");
/// Console.WriteLine(esValido); // True
/// 
/// // Caso inválido
/// bool esInvalido = validador.ValidarNIF("12345678");
/// Console.WriteLine(esInvalido); // False
/// </code>
/// </example>
public bool ValidarNIF(string nif)
{
    // Implementación
}
```

### 5.3 `<list>` - Listas Estructuradas

```csharp
/// <summary>
/// Valida los datos de un usuario según las reglas de negocio.
/// </summary>
/// <param name="usuario">Usuario a validar.</param>
/// <returns>Lista de errores de validación, vacía si no hay errores.</returns>
/// <remarks>
/// Las reglas de validación incluyen:
/// <list type="bullet">
/// <item>
/// <description>El nombre debe tener entre 2 y 50 caracteres.</description>
/// </item>
/// <item>
/// <description>El email debe tener formato válido.</description>
/// </item>
/// <item>
/// <description>La contraseña debe tener al menos 8 caracteres.</description>
/// </item>
/// <item>
/// <description>El teléfono debe tener formato internacional válido.</description>
/// </item>
/// </list>
/// </remarks>
public List<string> ValidarUsuario(Usuario usuario)
{
    // Implementación
}
```

### 5.4 `<paramref>` y `<see>` - Referencias Cruzadas

```csharp
/// <summary>
/// Combina dos listas eliminando duplicados.
/// </summary>
/// <param name="lista1">Primera lista a combinar.</param>
/// <param name="lista2">Segunda lista a combinar.</param>
/// <returns>
/// Lista combinada sin duplicados. Los elementos de <paramref name="lista1"/> 
/// aparecen primero, seguidos de los elementos únicos de <paramref name="lista2"/>.
/// </returns>
/// <remarks>
/// Este método utiliza <see cref="HashSet{T}"/> internamente para detectar duplicados.
/// Para objetos personalizados, asegúrese de implementar correctamente 
/// <see cref="Object.Equals(object)"/> y <see cref="Object.GetHashCode"/>.
/// </remarks>
/// <seealso cref="List{T}.Distinct"/>
public List<T> CombinarSinDuplicados<T>(List<T> lista1, List<T> lista2)
{
    // Implementación
}
```

### 5.5 `<inheritdoc>` - Herencia de Documentación

```csharp
/// <summary>
/// Interfaz base para todos los repositorios.
/// </summary>
public interface IRepositorioBase<T>
{
    /// <summary>
    /// Guarda los cambios pendientes en la base de datos.
    /// </summary>
    /// <returns>Número de entidades afectadas.</returns>
    Task<int> GuardarCambiosAsync();
}

/// <summary>
/// Repositorio específico para entidades de tipo Cliente.
/// </summary>
public class RepositorioClientes : IRepositorioBase<Cliente>
{
    /// <inheritdoc/>
    public async Task<int> GuardarCambiosAsync()
    {
        // Implementación específica
    }
    
    /// <summary>
    /// Busca clientes por nombre.
    /// </summary>
    /// <param name="nombre">Nombre a buscar.</param>
    /// <returns>Lista de clientes que coinciden con el nombre.</returns>
    public async Task<List<Cliente>> BuscarPorNombreAsync(string nombre)
    {
        // Implementación
    }
}
```

## 6. Documentación de Enumeraciones

### 6.1 Enumeración Básica

```csharp
/// <summary>
/// Define los estados posibles de un pedido en el sistema.
/// </summary>
public enum EstadoPedido
{
    /// <summary>
    /// El pedido ha sido creado pero no confirmado.
    /// </summary>
    Pendiente = 0,
    
    /// <summary>
    /// El pedido ha sido confirmado y está en proceso de preparación.
    /// </summary>
    EnProceso = 1,
    
    /// <summary>
    /// El pedido ha sido enviado al cliente.
    /// </summary>
    Enviado = 2,
    
    /// <summary>
    /// El pedido ha sido entregado al cliente.
    /// </summary>
    Entregado = 3,
    
    /// <summary>
    /// El pedido ha sido cancelado por el cliente o el sistema.
    /// </summary>
    Cancelado = 4
}
```

### 6.2 Enumeración con Flags

```csharp
/// <summary>
/// Define los permisos que puede tener un usuario en el sistema.
/// </summary>
/// <remarks>
/// Esta enumeración puede combinarse usando operadores bitwise para
/// asignar múltiples permisos a un usuario.
/// </remarks>
[Flags]
public enum Permisos
{
    /// <summary>
    /// Sin permisos asignados.
    /// </summary>
    Ninguno = 0,
    
    /// <summary>
    /// Permiso para leer datos.
    /// </summary>
    Leer = 1,
    
    /// <summary>
    /// Permiso para crear nuevos registros.
    /// </summary>
    Crear = 2,
    
    /// <summary>
    /// Permiso para actualizar registros existentes.
    /// </summary>
    Actualizar = 4,
    
    /// <summary>
    /// Permiso para eliminar registros.
    /// </summary>
    Eliminar = 8,
    
    /// <summary>
    /// Todos los permisos combinados.
    /// </summary>
    Todos = Leer | Crear | Actualizar | Eliminar
}
```

## 7. Documentación de Delegados y Eventos

### 7.1 Delegados

```csharp
/// <summary>
/// Representa un método que procesa un elemento y devuelve un resultado.
/// </summary>
/// <typeparam name="TInput">Tipo del elemento de entrada.</typeparam>
/// <typeparam name="TOutput">Tipo del resultado.</typeparam>
/// <param name="input">Elemento a procesar.</param>
/// <returns>Resultado del procesamiento.</returns>
public delegate TOutput Procesador<TInput, TOutput>(TInput input);
```

### 7.2 Eventos

```csharp
/// <summary>
/// Proporciona datos para el evento de cliente actualizado.
/// </summary>
public class ClienteActualizadoEventArgs : EventArgs
{
    /// <summary>
    /// Obtiene el cliente que fue actualizado.
    /// </summary>
    public Cliente Cliente { get; }
    
    /// <summary>
    /// Obtiene la fecha y hora de la actualización.
    /// </summary>
    public DateTime FechaActualizacion { get; }
    
    /// <summary>
    /// Inicializa una nueva instancia de <see cref="ClienteActualizadoEventArgs"/>.
    /// </summary>
    /// <param name="cliente">Cliente actualizado.</param>
    public ClienteActualizadoEventArgs(Cliente cliente)
    {
        Cliente = cliente;
        FechaActualizacion = DateTime.UtcNow;
    }
}
```

## 8. Mejores Prácticas

### 8.1 Qué Documentar

**SÍ documentar:**
- Clases, interfaces, estructuras y enumeraciones públicas
- Métodos públicos y protegidos
- Propiedades públicas y protegidas
- Parámetros y valores de retorno
- Excepciones que pueden lanzarse
- Comportamientos no obvios o complejos
- Precondiciones y postcondiciones
- Efectos secundarios importantes
- APIs públicas y contratos

**NO documentar en exceso:**
- Implementaciones privadas obvias
- Getters/setters triviales sin lógica
- Métodos con nombres autoexplicativos y sin complejidad
- Detalles de implementación que pueden cambiar

### 8.2 Calidad de la Documentación

**Buena documentación:**

```csharp
/// <summary>
/// Calcula el precio final aplicando descuentos por volumen.
/// </summary>
/// <param name="precioUnitario">Precio por unidad del producto.</param>
/// <param name="cantidad">Cantidad de unidades a comprar.</param>
/// <returns>
/// Precio total con descuentos aplicados. 
/// Descuentos: 5% para 10-49 unidades, 10% para 50-99, 15% para 100+.
/// </returns>
public decimal CalcularPrecioConDescuento(decimal precioUnitario, int cantidad)
{
    // Implementación
}
```

**Mala documentación:**

```csharp
/// <summary>
/// Calcula el precio.
/// </summary>
/// <param name="p">Precio.</param>
/// <param name="c">Cantidad.</param>
/// <returns>El precio.</returns>
public decimal CalcularPrecioConDescuento(decimal p, int c)
{
    // Implementación
}
```

### 8.3 Lenguaje y Estilo

- Use lenguaje claro y conciso
- Escriba en tercera persona del presente: "Obtiene", "Calcula", "Valida"
- Sea específico y preciso
- Evite redundancias obvias
- Mantenga consistencia en el estilo
- Use terminología del dominio correctamente

### 8.4 Mantenimiento

```csharp
/// <summary>
/// Procesa un pago mediante tarjeta de crédito.
/// </summary>
/// <param name="monto">Monto a cobrar en euros.</param>
/// <param name="tarjeta">Información de la tarjeta de crédito.</param>
/// <returns>Resultado del procesamiento del pago.</returns>
/// <remarks>
/// IMPORTANTE: Este método fue actualizado el 2024-10-15 para soportar
/// autenticación 3D Secure 2.0. Versiones anteriores usaban 3DS 1.0.
/// </remarks>
/// <exception cref="PagoRechazadoException">
/// Se lanza cuando la transacción es rechazada por el banco.
/// </exception>
public ResultadoPago ProcesarPago(decimal monto, TarjetaCredito tarjeta)
{
    // Implementación
}
```

## 9. Generación de Documentación

### 9.1 Habilitar Generación de XML

En el archivo `.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\$(AssemblyName).xml</DocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn> <!-- Suprime warnings por miembros sin documentar -->
  </PropertyGroup>
</Project>
```

### 9.2 Herramientas de Generación

**DocFX:**
```bash
# Instalación
dotnet tool install -g docfx

# Generación de documentación
docfx init
docfx build
docfx serve
```

**Sandcastle:**
- Genera documentación estilo MSDN
- Soporta múltiples formatos de salida
- Integración con Visual Studio

## 10. Relación con Testing

### 10.1 Documentación para Tests

```csharp
/// <summary>
/// Valida que un email tenga formato correcto según RFC 5322.
/// </summary>
/// <param name="email">Email a validar.</param>
/// <returns>True si el formato es válido; false en caso contrario.</returns>
/// <remarks>
/// Casos de prueba recomendados:
/// <list type="bullet">
/// <item><description>Email válido estándar: usuario@dominio.com</description></item>
/// <item><description>Email con subdominios: usuario@mail.dominio.com</description></item>
/// <item><description>Email con caracteres especiales: usuario+tag@dominio.com</description></item>
/// <item><description>Email inválido sin @: usuariodominio.com</description></item>
/// <item><description>Email inválido sin dominio: usuario@</description></item>
/// <item><description>Cadena vacía o null</description></item>
/// </list>
/// </remarks>
public bool ValidarEmail(string email)
{
    // Implementación
}
```

### 10.2 Documentación de Contratos

```csharp
/// <summary>
/// Divide dos números decimales.
/// </summary>
/// <param name="dividendo">Número a dividir.</param>
/// <param name="divisor">Número por el cual dividir. No puede ser cero.</param>
/// <returns>Resultado de la división.</returns>
/// <exception cref="DivideByZeroException">
/// Se lanza cuando <paramref name="divisor"/> es cero.
/// </exception>
/// <remarks>
/// Precondiciones:
/// - divisor != 0
/// 
/// Postcondiciones:
/// - El resultado es exacto con precisión decimal
/// - No se pierden decimales significativos
/// </remarks>
public decimal Dividir(decimal dividendo, decimal divisor)
{
    if (divisor == 0)
        throw new DivideByZeroException("No se puede dividir por cero.");
    
    return dividendo / divisor;
}
```

## 11. Ejercicios Prácticos

### Ejercicio 1: Documentar una Clase Completa

Documente completamente la siguiente clase:

```csharp
public class CarritoCompras
{
    private List<ItemCarrito> _items = new List<ItemCarrito>();
    
    public void AgregarItem(Producto producto, int cantidad)
    {
        // Implementación
    }
    
    public decimal ObtenerTotal()
    {
        // Implementación
    }
    
    public bool AplicarCupon(string codigoCupon)
    {
        // Implementación
    }
}
```

### Ejercicio 2: Documentar una Interface

Documente la siguiente interface siguiendo las mejores prácticas:

```csharp
public interface IServicioEmail
{
    Task<bool> EnviarEmailAsync(string destinatario, string asunto, string cuerpo);
    Task<bool> EnviarEmailConAdjuntosAsync(string destinatario, string asunto, string cuerpo, List<Adjunto> adjuntos);
    Task<List<Email>> ObtenerEmailsRecibidosAsync(DateTime desde, DateTime hasta);
}
```

### Ejercicio 3: Mejorar Documentación Existente

Mejore la siguiente documentación deficiente:

```csharp
/// <summary>
/// Hace algo con los datos.
/// </summary>
/// <param name="datos">Los datos.</param>
/// <returns>Un resultado.</returns>
public Result ProcesarDatos(Data datos)
{
    // Implementación
}
```

## 12. Checklist de Documentación

Al documentar código .NET, verifique que:

- [ ] Todas las clases públicas tienen `<summary>`
- [ ] Todos los métodos públicos documentan sus parámetros
- [ ] Los valores de retorno están explicados claramente
- [ ] Las excepciones están documentadas con `<exception>`
- [ ] Se usan `<remarks>` cuando hay información adicional relevante
- [ ] Los ejemplos de uso están presentes en APIs complejas
- [ ] Las referencias cruzadas usan `<see>` y `<seealso>`
- [ ] Los parámetros genéricos tienen `<typeparam>`
- [ ] La documentación está en español (o el idioma del proyecto)
- [ ] No hay información redundante o innecesaria
- [ ] La terminología es consistente en todo el proyecto
- [ ] Se ha generado el archivo XML correctamente

## Recursos Adicionales

- [Microsoft Docs - XML Documentation Comments](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/)
- [DocFX Documentation](https://dotnet.github.io/docfx/)
- [Sandcastle Help File Builder](https://github.com/EWSoftware/SHFB)

---

**Nota:** Esta guía forma parte del curso "Testing .NET y QA (Avanzado)" y debe usarse como referencia para mantener estándares de documentación profesionales en proyectos .NET.

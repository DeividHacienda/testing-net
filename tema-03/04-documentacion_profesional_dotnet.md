# Documentación Profesional de Código .NET

## 1. Mejores prácticas: qué documentar y qué no

### ✅ QUÉ SÍ DOCUMENTAR

**APIs públicas y contratos**
```csharp
/// <summary>
/// Procesa un pago con tarjeta de crédito.
/// </summary>
/// <param name="amount">Monto a cobrar en la moneda configurada.</param>
/// <param name="cardToken">Token de la tarjeta proporcionado por el gateway.</param>
/// <returns>ID de la transacción si fue exitosa.</returns>
/// <exception cref="PaymentException">Si el pago es rechazado.</exception>
public async Task<string> ProcessPayment(decimal amount, string cardToken)
{
    // implementación
}
```

**Comportamientos no obvios o efectos secundarios**
```csharp
/// <summary>
/// Invalida la sesión del usuario.
/// </summary>
/// <remarks>
/// IMPORTANTE: Este método también elimina todos los tokens de refresh 
/// asociados y notifica a los servicios externos.
/// </remarks>
public void InvalidateSession(int userId)
{
    // implementación
}
```

**Parámetros con restricciones o formatos específicos**
```csharp
/// <summary>
/// Valida un número de teléfono español.
/// </summary>
/// <param name="phone">Número en formato +34XXXXXXXXX o 9 dígitos.</param>
public bool ValidatePhone(string phone)
{
    // implementación
}
```

### ❌ QUÉ NO DOCUMENTAR

**Código auto-explicativo**
```csharp
// ❌ MAL - documentación redundante
/// <summary>
/// Obtiene el nombre del usuario.
/// </summary>
public string GetUserName() => _userName;

// ✅ BIEN - el nombre del método es suficiente
public string GetUserName() => _userName;
```

**Detalles de implementación interna**
```csharp
// ❌ MAL - expone detalles internos
/// <summary>
/// Calcula descuento. Usa un Dictionary interno para cachear resultados.
/// </summary>
private decimal CalculateDiscount(Order order)

// ✅ BIEN - solo documenta lo relevante
/// <summary>
/// Calcula el descuento aplicable según las reglas de negocio.
/// </summary>
private decimal CalculateDiscount(Order order)
```

**Comentarios obvios o que repiten el código**
```csharp
// ❌ MAL
// Incrementa el contador
counter++;

// ✅ BIEN - sin comentario innecesario
counter++;
```

---

## 2. Documentación de API públicas vs. código interno

### API PÚBLICA (Siempre documentar)

```csharp
/// <summary>
/// Servicio para gestionar operaciones de usuarios.
/// </summary>
public interface IUserService
{
    /// <summary>
    /// Crea un nuevo usuario en el sistema.
    /// </summary>
    /// <param name="email">Email único del usuario.</param>
    /// <param name="password">Contraseña (mínimo 8 caracteres).</param>
    /// <returns>El usuario creado con su ID asignado.</returns>
    /// <exception cref="DuplicateEmailException">Si el email ya existe.</exception>
    Task<User> CreateUser(string email, string password);
    
    /// <summary>
    /// Desactiva una cuenta de usuario.
    /// </summary>
    /// <remarks>
    /// El usuario no podrá iniciar sesión pero sus datos se mantienen.
    /// </remarks>
    Task DeactivateUser(int userId);
}
```

**Razón:** Otros desarrolladores consumirán esta API. Necesitan saber exactamente qué hace, qué espera y qué devuelve.

### CÓDIGO INTERNO (Documentar selectivamente)

```csharp
public class UserService : IUserService
{
    // ✅ Documentar si hay lógica compleja
    /// <summary>
    /// Valida que la contraseña cumpla los requisitos de seguridad.
    /// </summary>
    /// <remarks>
    /// Requisitos: 8+ caracteres, 1 mayúscula, 1 número, 1 símbolo.
    /// </remarks>
    private bool ValidatePassword(string password)
    {
        // implementación compleja
    }
    
    // ❌ No documentar métodos simples internos
    private string HashPassword(string password)
    {
        return BCrypt.HashPassword(password);
    }
    
    // ✅ Documentar si hay efectos secundarios importantes
    /// <summary>
    /// Envía email de bienvenida y registra evento en analytics.
    /// </summary>
    private async Task SendWelcomeEmail(User user)
    {
        // implementación
    }
}
```

### COMPARACIÓN PRÁCTICA

| Aspecto | API Pública | Código Interno |
|---------|-------------|----------------|
| **Documentación** | Obligatoria y completa | Selectiva, solo lo complejo |
| **Nivel de detalle** | Alto | Medio-bajo |
| **Audiencia** | Consumidores externos | Equipo interno |
| **Estabilidad** | Alta (breaking changes evitados) | Flexible |

---

## 3. Relación entre documentación y testeabilidad

### PRINCIPIO CLAVE
**Código difícil de documentar = Código difícil de testear**

### EJEMPLO 1: Método con demasiadas responsabilidades

```csharp
// ❌ DIFÍCIL DE DOCUMENTAR Y TESTEAR
/// <summary>
/// Procesa el pedido... y valida stock... y calcula envío... 
/// y envía emails... y actualiza inventario...
/// </summary>
public void ProcessOrder(Order order)
{
    // 200 líneas de código
    // Múltiples responsabilidades
}
```

**Problema:** Si te cuesta escribir el `<summary>`, el método hace demasiado.

```csharp
// ✅ FÁCIL DE DOCUMENTAR Y TESTEAR
/// <summary>
/// Valida que hay stock suficiente para el pedido.
/// </summary>
public bool ValidateStock(Order order)
{
    // lógica específica
}

/// <summary>
/// Calcula el costo de envío según destino y peso.
/// </summary>
public decimal CalculateShipping(Order order)
{
    // lógica específica
}
```

### EJEMPLO 2: Dependencias ocultas

```csharp
// ❌ DIFÍCIL DE DOCUMENTAR (dependencias ocultas)
/// <summary>
/// Envía notificación al usuario.
/// </summary>
/// <remarks>
/// NOTA: Requiere que exista un archivo config.json en la raíz...
/// </remarks>
public void NotifyUser(int userId)
{
    var config = File.ReadAllText("config.json"); // ⚠️ Dependencia oculta
    var smtpServer = JsonSerializer.Deserialize<Config>(config).SmtpServer;
    // ...
}
```

**Test problemático:**
```csharp
[Test]
public void NotifyUser_Should_SendEmail()
{
    // ⚠️ Necesito crear un archivo físico para testear
    File.WriteAllText("config.json", "...");
    
    _service.NotifyUser(123);
    
    // Difícil de verificar si se envió el email
}
```

```csharp
// ✅ FÁCIL DE DOCUMENTAR (dependencias explícitas)
/// <summary>
/// Envía notificación al usuario usando el servicio de email configurado.
/// </summary>
public void NotifyUser(int userId, IEmailService emailService)
{
    emailService.Send(userId, "mensaje");
}
```

**Test sencillo:**
```csharp
[Test]
public void NotifyUser_Should_SendEmail()
{
    var mockEmail = new Mock<IEmailService>();
    
    _service.NotifyUser(123, mockEmail.Object);
    
    mockEmail.Verify(e => e.Send(123, It.IsAny<string>()), Times.Once);
}
```

### SEÑALES DE ALERTA

| Si al documentar encuentras... | Indica problema de testeabilidad |
|-------------------------------|----------------------------------|
| Muchos efectos secundarios no relacionados | Violación de Single Responsibility |
| Dependencias difíciles de explicar | Acoplamiento fuerte |
| Demasiadas excepciones posibles | Complejidad excesiva |
| "Esto solo funciona si..." | Dependencias ocultas |

### CHECKLIST: Código documentable = Código testeable

- ✅ **Un propósito claro** → Puedes escribir un `<summary>` en una línea
- ✅ **Parámetros bien definidos** → Cada `<param>` tiene sentido independiente
- ✅ **Retorno predecible** → El `<returns>` describe un resultado claro
- ✅ **Excepciones controladas** → Cada `<exception>` tiene una causa específica
- ✅ **Sin efectos secundarios ocultos** → Los `<remarks>` no dicen "IMPORTANTE: esto también..."

---

## Resumen ejecutivo

1. **Documenta lo público, complejo o no obvio**. No pierdas tiempo en código autoexplicativo.

2. **APIs públicas = documentación exhaustiva**. Código interno = documentación selectiva.

3. **Si es difícil documentar, será difícil testear**. Usa la documentación como indicador de calidad del diseño.

**Regla de oro:** Si necesitas más de 3 líneas para explicar qué hace un método, probablemente hace demasiado.

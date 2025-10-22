# ATDD en la Práctica Real

## La Confusión Común sobre ATDD

**Muchos piensan:**
```
ATDD = El cliente ejecuta pruebas técnicas
```

**La realidad es:**
```
ATDD = Definir CRITERIOS con el cliente ANTES de desarrollar
       Desarrollar con pruebas automatizadas
       Demostrar la aplicación TERMINADA al cliente
```

---

## El Proceso Real de ATDD

### Fase 1: Definición (ANTES de programar)

**Reunión con el cliente:**

```
Tú: "Cuéntame qué necesitas con los descuentos"

Cliente: "Quiero que cuando compren mucho, tengan descuento"

Tú: "Ok, ¿a partir de cuántas unidades?"

Cliente: "Mmm, a partir de 10 unidades"

Tú: "¿Qué porcentaje de descuento?"

Cliente: "Un 10%"

Tú: "¿Y si compran más de 50?"

Cliente: "Ah, entonces 20%"

Tú: "Perfecto, entonces quedaría así:
     - Menos de 10: sin descuento
     - 10 a 49: 10% descuento  
     - 50 o más: 20% descuento
     ¿Es correcto?"

Cliente: "Sí, perfecto"
```

**Documento de Criterios (lenguaje simple):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCIONALIDAD: Descuentos por Volumen
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✓ Si compra 1-9 unidades: NO hay descuento
✓ Si compra 10-49 unidades: 10% de descuento
✓ Si compra 50 o más: 20% de descuento
✓ El descuento debe verse en pantalla
✓ Debe funcionar en el carrito de compras

Firmado: _____________ (Cliente)
Fecha: _______________
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Esto es lo que firma el cliente ANTES de que programes nada.**

---

### Fase 2: Desarrollo (TÚ SOLO)

Ahora sí, tú como desarrollador usas TDD/ATDD:

```csharp
// Test 1 - Basado en el criterio acordado
[TestMethod]
public void ComprarMenosDe10_SinDescuento()
{
    // Este test lo escribes TÚ
    // El cliente NO lo ve
    var carrito = new CarritoCompras();
    carrito.AgregarProducto(new Producto { Precio = 100m }, 5);
    
    Assert.AreEqual(500m, carrito.Total);
    Assert.AreEqual(0m, carrito.Descuento);
}

// Test 2
[TestMethod]
public void Comprar15Unidades_Descuento10Porciento()
{
    var carrito = new CarritoCompras();
    carrito.AgregarProducto(new Producto { Precio = 100m }, 15);
    
    Assert.AreEqual(1500m, carrito.Subtotal);
    Assert.AreEqual(150m, carrito.Descuento);
    Assert.AreEqual(1350m, carrito.Total);
}

// Test 3
[TestMethod]
public void Comprar60Unidades_Descuento20Porciento()
{
    var carrito = new CarritoCompras();
    carrito.AgregarProducto(new Producto { Precio = 100m }, 60);
    
    Assert.AreEqual(6000m, carrito.Subtotal);
    Assert.AreEqual(1200m, carrito.Descuento);
    Assert.AreEqual(4800m, carrito.Total);
}
```

**Implementas el código:**
```csharp
public class CarritoCompras
{
    public decimal Subtotal { get; private set; }
    public decimal Descuento { get; private set; }
    public decimal Total { get; private set; }
    
    public void AgregarProducto(Producto producto, int cantidad)
    {
        // Implementación siguiendo TDD
        Subtotal = producto.Precio * cantidad;
        
        if (cantidad >= 50)
            Descuento = Subtotal * 0.20m;
        else if (cantidad >= 10)
            Descuento = Subtotal * 0.10m;
        else
            Descuento = 0m;
            
        Total = Subtotal - Descuento;
    }
}
```

**Ejecutas tus pruebas:**
```
✅ ComprarMenosDe10_SinDescuento
✅ Comprar15Unidades_Descuento10Porciento
✅ Comprar60Unidades_Descuento20Porciento

Todas las pruebas pasan → Código listo
```

**Desarrollas la interfaz:**
- Página del carrito
- Formulario para agregar productos
- Mostrar subtotal, descuento, total
- Responsive, bonito, etc.

---

### Fase 3: Demostración (CON el cliente)

**Ahora sí llamas al cliente:**

```
Email:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Asunto: Listo para revisar - Descuentos por Volumen

Hola [Cliente],

Ya está lista la funcionalidad de descuentos que 
acordamos. ¿Cuándo podemos tener una reunión de 
15 minutos para que lo veas funcionando?

Saludos,
[Tu nombre]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**En la reunión (aplicación TERMINADA):**

```
Tú: "¿Recuerdas los criterios que acordamos?"

[Proyectas el documento firmado]

Cliente: "Sí, los descuentos por volumen"

Tú: "Perfecto, vamos a verificar cada punto"

[Abres la aplicación web FUNCIONANDO]

Tú: "Primer caso: compra de 5 unidades"

[Agregas 5 productos, el cliente VE la pantalla]

Cliente: "Bien, $500, sin descuento ✓"

Tú: "Segundo caso: 15 unidades"

[Cambias a 15 unidades]

Cliente: "Perfecto, veo el 10% de descuento ✓"

Tú: "Último caso: 60 unidades"

[Cambias a 60 unidades]

Cliente: "Excelente, 20% de descuento ✓"

Tú: "¿Es lo que necesitabas?"

Cliente: "Sí, aprobado. ¿Cuándo se sube a producción?"
```

---

## Diferencia Clave: ATDD vs Waterfall

### Waterfall Tradicional:
```
1. Cliente pide "descuentos"
2. Desarrollas 2 semanas
3. Le muestras al cliente
4. Cliente: "No, yo quería que fuera diferente"
5. Re-trabajo de 1 semana
6. Le muestras de nuevo
7. Cliente: "Ahora sí"

Total: 3 semanas + frustración
```

### Con ATDD:
```
1. Reunión 30 min: Definir criterios EXACTOS
2. Cliente FIRMA los criterios
3. Desarrollas 1 semana con TDD (confianza 100%)
4. Le muestras al cliente
5. Cliente: "Exactamente como acordamos"

Total: 1 semana + documento firmado
```

---

## ¿Qué Pasa si el Cliente Cambia de Opinión?

**Durante la demo:**

```
Cliente: "Hmm, viéndolo ahora, creo que el 
         descuento debería ser del 15%, no 10%"

Tú: "Sin problema. Eso sería un cambio a los 
     criterios que acordamos aquí [muestras 
     documento]. ¿Quieres que lo ajustemos?"

Cliente: "Sí, por favor"

Tú: "Ok, actualizo los criterios, lo desarrollo 
     y te muestro en 2 días. ¿Te parece?"

Cliente: "Perfecto"
```

**Lo que haces por detrás:**
```csharp
// Actualizas el test (5 minutos)
[TestMethod]
public void Comprar15Unidades_Descuento15Porciento()
{
    var carrito = new CarritoCompras();
    carrito.AgregarProducto(new Producto { Precio = 100m }, 15);
    
    Assert.AreEqual(1500m, carrito.Subtotal);
    Assert.AreEqual(225m, carrito.Descuento);  // Cambió
    Assert.AreEqual(1275m, carrito.Total);     // Cambió
}

// Actualizas el código (5 minutos)
if (cantidad >= 10)
    Descuento = Subtotal * 0.15m;  // Era 0.10m

// Ejecutas las pruebas
✅ Todas pasan

// Despliegas
```

**Total: 15 minutos de cambio gracias a las pruebas automáticas**

---

## Tipos de Cliente y Estrategias de Demostración

### 1. Cliente NO Técnico (Mayoría de los casos)

**❌ NO hacer:**
- Mostrar código
- Ejecutar test runners
- Hablar de "assertions" o "mocks"
- Mostrar Visual Studio

**✅ SÍ hacer:**
- Mostrar solo la aplicación funcionando
- Usar lenguaje de negocio puro
- Crear un checklist visual simple
- Demostrar con datos reales del cliente

**Ejemplo de checklist visual:**

```
┌────────────────────────────────────────────────┐
│ DEMOSTRACIÓN - Descuentos por Volumen         │
├────────────────────────────────────────────────┤
│                                                │
│ ☐ Compra pequeña (5 unidades)                 │
│ ☐ Compra mediana (15 unidades) → 10% desc.    │
│ ☐ Compra grande (60 unidades) → 20% desc.     │
│ ☐ Ver desglose en pantalla                    │
│                                                │
└────────────────────────────────────────────────┘
```

---

### 2. Cliente Técnico (Product Owner, CTO)

**Puedes mostrar:**
- Test runner pasando pruebas
- Explicación breve de cada escenario
- Cobertura de código (como indicador de calidad)
- Aplicación funcionando

**Formato híbrido:**
```
1. Mostrar checklist de criterios
2. Ejecutar suite de pruebas (2 minutos)
3. Demostrar aplicación (10 minutos)
4. Mostrar métricas de calidad
```

---

### 3. Cliente Detallista/Escéptico

**Estrategia:**
- Documento con screenshots de "antes y después"
- Video pregrabado de los escenarios
- Sesión más larga con tiempo para preguntas
- Permitirle a ELLOS usar la aplicación

**Script:**
```
Tú: "Te voy a dar acceso a esta versión de prueba. 
     Aquí está la lista de lo que desarrollamos.
     Prueba tú mismo cada escenario."

[Le das acceso al ambiente de QA]

Cliente: [Prueba por su cuenta]

Cliente: "¿Qué pasa si agrego 49 unidades y luego 
         agrego 1 más?"

Tú: "Excelente pregunta, pruébalo..."

Cliente: [Prueba] "Ah, cambia automáticamente al 20%"

Tú: "Exacto, eso fue parte de los casos que probamos."
```

---

## Herramientas Visuales para Clientes No Técnicos

### A) Checklist Interactivo en PowerPoint/Excel

```
Criterio                        Estado    Notas
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Compra 5 unidades               [ ✓ ]     OK
Compra 15 unidades              [ ✓ ]     OK  
Compra 60 unidades              [ ✓ ]     OK
Descuento visible               [ ✓ ]     OK
Funciona en móvil               [   ]     Pendiente
```

### B) Video Pregrabado

```
Opción 1: Grabar pantalla con narración
  "Hola, aquí está la funcionalidad terminada.
   Primero verás el caso de 5 unidades..."

Ventajas:
  - El cliente lo ve cuando quiera
  - Puedes editarlo y pulirlo
  - No hay nervios en vivo
  - Sirve como documentación
```

### C) Ambiente de Pruebas con Acceso Directo

```
Email al cliente:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Hola [Nombre],

Ya está lista la funcionalidad de descuentos.

🔗 Pruébalo aquí: https://demo.tuapp.com
   Usuario: cliente@demo.com
   Clave: Demo123

📋 Checklist de prueba:
   ☐ Agrega 5 productos → Sin descuento
   ☐ Agrega 15 productos → 10% descuento
   ☐ Agrega 60 productos → 20% descuento

Por favor pruébalo y dime si es lo que necesitas.

Saludos,
[Tu nombre]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Formato Simplificado: "Story Mapping Demo"

Para clientes completamente no técnicos:

```
┌─────────────────────────────────────────────┐
│          HISTORIA: Descuentos               │
├─────────────────────────────────────────────┤
│                                             │
│  📋 Lo que pediste:                         │
│     "Quiero que las compras grandes         │
│      tengan descuento automático"           │
│                                             │
│  ✅ Lo que construimos:                     │
│     → Compras 1-9: Sin descuento            │
│     → Compras 10-49: 10% descuento          │
│     → Compras 50+: 20% descuento            │
│                                             │
│  👀 Ahora lo verás funcionar...             │
│                                             │
└─────────────────────────────────────────────┘
```

**Después muestras la aplicación, punto.**

---

## Las Pruebas de Aceptación "Por Detrás"

### Lo que el cliente NO ve pero TÚ sí usas:

```csharp
// Antes de la demo, ejecutas esto:
[TestClass]
public class PreDemoValidation
{
    [TestMethod]
    public void ValidarTodosLosEscenariosParaDemo()
    {
        // Escenario 1: 5 unidades
        var resultado1 = ProcesarCompra(5);
        Assert.AreEqual(0, resultado1.Descuento);
        
        // Escenario 2: 15 unidades
        var resultado2 = ProcesarCompra(15);
        Assert.AreEqual(0.10m, resultado2.PorcentajeDescuento);
        
        // Escenario 3: 60 unidades
        var resultado3 = ProcesarCompra(60);
        Assert.AreEqual(0.20m, resultado3.PorcentajeDescuento);
    }
    
    private ResultadoCompra ProcesarCompra(int cantidad)
    {
        var carrito = new CarritoCompras();
        carrito.AgregarProducto(new Producto { Precio = 100m }, cantidad);
        return new ResultadoCompra 
        { 
            Descuento = carrito.Descuento,
            PorcentajeDescuento = carrito.Descuento / carrito.Subtotal
        };
    }
}
```

**Si todas pasan → Confianza al 100% para la demo**

El cliente nunca ve este código, pero tú **sabes** que todo funciona.

---

## Resumen Visual

### Lo que el Cliente VE:

```
┌─────────────────────────────────────────────────┐
│ LO QUE EL CLIENTE VE:                          │
├─────────────────────────────────────────────────┤
│ 1. Reunión de requisitos (criterios escritos)  │
│ 2. [Espera mientras desarrollas]               │
│ 3. Aplicación terminada funcionando            │
│ 4. Aprueba o pide ajustes                      │
└─────────────────────────────────────────────────┘
```

### Lo que el Cliente NO VE:

```
┌─────────────────────────────────────────────────┐
│ LO QUE EL CLIENTE NO VE:                       │
├─────────────────────────────────────────────────┤
│ - Tus pruebas automatizadas                     │
│ - Tu proceso TDD                                │
│ - Visual Studio                                 │
│ - Test runners                                  │
│ - Código fuente                                 │
└─────────────────────────────────────────────────┘
```

---

## Ventaja Real de ATDD

### Sin ATDD:
```
Desarrollador: "Terminé los descuentos"
Cliente: "Pero yo pensé que sería..."
Desarrollador: "Ah, no me dijiste eso"
→ Conflicto y re-trabajo
```

### Con ATDD:
```
Desarrollador: "Aquí está lo que acordamos" [muestra documento]
Cliente: "Ah cierto, sí lo acordamos así"
Cliente: "¿Podemos cambiar esto?"
Desarrollador: "Claro, actualizo y te muestro mañana"
→ Sin conflictos, cambios controlados
```

---

## Niveles de Demostración

```
Nivel 1 - Cliente No Técnico (90% de los casos)
└─ Solo aplicación + checklist visual

Nivel 2 - Cliente Semi-Técnico
└─ Aplicación + mención de "pruebas automatizadas"

Nivel 3 - Cliente Técnico
└─ Tests + Aplicación + Métricas

Nivel 4 - Equipo Técnico Interno
└─ Full ATDD con código y todo
```

---

## Frase Clave para Clientes No Técnicos

Cuando te pregunten sobre "pruebas de aceptación":

> "Las pruebas de aceptación son simplemente verificar que lo que construimos hace exactamente lo que necesitas. Vamos a probarlo juntos punto por punto."

**NO digas:** "Vamos a ejecutar la suite de tests de aceptación automatizados con MSTest"

**SÍ di:** "Vamos a revisar que todo funcione como quedamos"

---

## Conclusión Final

### ATDD NO es:
- Enseñar al cliente a programar
- Mostrar código al cliente
- Que el cliente ejecute pruebas técnicas

### ATDD ES:
- Definir criterios claros ANTES
- Desarrollar con confianza usando pruebas automáticas
- Demostrar aplicación FUNCIONANDO
- Tener evidencia documentada de lo acordado

---

## El Flujo Completo en Resumen

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  FASE 1: DEFINICIÓN                                  │
│  ┌────────────────────────────────────────┐          │
│  │ Reunión con cliente                    │          │
│  │ → Escribir criterios en lenguaje simple│          │
│  │ → Cliente FIRMA los criterios          │          │
│  └────────────────────────────────────────┘          │
│                    ↓                                 │
│  FASE 2: DESARROLLO (TÚ SOLO)                        │
│  ┌────────────────────────────────────────┐          │
│  │ Escribir tests basados en criterios    │          │
│  │ → Implementar código con TDD           │          │
│  │ → Desarrollar interfaz                 │          │
│  │ → Ejecutar pruebas (todas ✅)          │          │
│  └────────────────────────────────────────┘          │
│                    ↓                                 │
│  FASE 3: DEMOSTRACIÓN (CON CLIENTE)                  │
│  ┌────────────────────────────────────────┐          │
│  │ Mostrar aplicación TERMINADA           │          │
│  │ → Verificar cada criterio              │          │
│  │ → Cliente aprueba o solicita cambios   │          │
│  └────────────────────────────────────────┘          │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Beneficios Tangibles

| Aspecto | Sin ATDD | Con ATDD |
|---------|----------|----------|
| **Tiempo de desarrollo** | 3 semanas + retrabajos | 1 semana |
| **Malentendidos** | Frecuentes | Mínimos |
| **Confianza del equipo** | Baja | Alta |
| **Documentación** | Desactualizada | Viva y ejecutable |
| **Cambios** | Costosos y arriesgados | Rápidos y seguros |
| **Satisfacción del cliente** | Variable | Alta |

---

**Recuerda:** El cliente solo ve: **Requisitos → Aplicación terminada → Aprobación**

Tú usas las pruebas automatizadas para tener **seguridad al 100%** de que cumples los criterios, pero eso es tu herramienta interna profesional.

**Las pruebas de aceptación son tu red de seguridad técnica, pero la demostración es una conversación de negocio.**

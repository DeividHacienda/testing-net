# Pruebas de cobertura y métricas de código en .NET 4.8

## 1. Introducción a la cobertura de código

La cobertura de código es una métrica que mide el porcentaje de código fuente ejecutado durante las pruebas. No garantiza ausencia de errores, pero identifica código no probado.

### Tipos de cobertura

**Cobertura de líneas**: Porcentaje de líneas ejecutadas.

**Cobertura de ramas**: Porcentaje de caminos condicionales evaluados (if/else, switch).

**Cobertura de métodos**: Porcentaje de métodos invocados.

### Ejemplo básico

```csharp
public class Calculator
{
    public int Divide(int a, int b)
    {
        if (b == 0)
            throw new ArgumentException("Division por cero");
        
        return a / b;
    }
}

// Prueba con cobertura parcial (solo cubre el caso normal)
[TestMethod]
public void Divide_NumerosValidos_RetornaResultado()
{
    var calc = new Calculator();
    Assert.AreEqual(5, calc.Divide(10, 2));
}
// Cobertura: 66% (no prueba la excepción)
```

## 2. Medición de cobertura con herramientas nativas de Visual Studio

Visual Studio Professional y Enterprise incluyen herramientas integradas de cobertura de código para .NET 4.8.

### Características principales

- Integración nativa con IDE
- Resaltado visual de código cubierto/no cubierto
- Reportes detallados por ensamblado, clase y método
- Soporte para MSTest, NUnit y xUnit

### Uso en Visual Studio

1. **Ejecutar con cobertura**: Menú Test → Analyze Code Coverage → All Tests
2. **Visualizar resultados**: Ventana "Code Coverage Results"
3. **Navegación**: Doble clic en resultados para ver código coloreado

```csharp
// Código con cobertura visualizada
public decimal CalcularDescuento(decimal precio, int cantidad)
{
    if (cantidad >= 10)  // Verde: cubierto
        return precio * 0.9m;
    else if (cantidad >= 5)  // Rojo: no cubierto
        return precio * 0.95m;
    
    return precio;  // Verde: cubierto
}
```

### Limitaciones

- Solo disponible en versiones Professional/Enterprise
- No funciona con .NET Core sin adaptaciones
- Reportes limitados en formato HTML/XML

## 3. OpenCover para .NET Framework 4.8

OpenCover es una herramienta gratuita y open-source específica para .NET Framework que genera reportes de cobertura.

### Instalación

```bash
# Instalar vía NuGet en proyecto de pruebas
Install-Package OpenCover -Version 4.7.1221

# O descargar ejecutable independiente
```

### Configuración básica

```xml
<!-- packages.config -->
<package id="OpenCover" version="4.7.1221" targetFramework="net48" />
<package id="ReportGenerator" version="4.8.13" targetFramework="net48" />
```

### Ejecución desde línea de comandos

```bash
# Ejecutar pruebas con OpenCover
OpenCover.Console.exe ^
  -target:"C:\Program Files\Microsoft Visual Studio\2022\Professional\Common7\IDE\MSTest.exe" ^
  -targetargs:"/testcontainer:MiProyecto.Tests.dll" ^
  -filter:"+[MiProyecto*]* -[*Tests]*" ^
  -output:coverage.xml ^
  -register:user

# Generar reporte HTML
ReportGenerator.exe ^
  -reports:coverage.xml ^
  -targetdir:coverage-report ^
  -reporttypes:Html
```

### Ejemplo de filtros

```bash
# Incluir solo namespaces principales, excluir tests y modelos generados
-filter:"+[MiApp.Business]* +[MiApp.Services]* -[*.Tests]* -[*]*Model"
```

### Ventajas

- Gratuito y compatible con .NET 4.8
- Genera reportes detallados en XML
- Integrable con ReportGenerator para reportes visuales

### Desventajas

- Requiere configuración manual
- Sintaxis de línea de comandos compleja
- Menor rendimiento que herramientas modernas

## 4. Coverlet (compatibilidad con .NET 4.8)

Aunque Coverlet está diseñado principalmente para .NET Core, puede usarse con .NET 4.8 mediante MSBuild.

### Instalación

```xml
<!-- En archivo .csproj del proyecto de pruebas -->
<ItemGroup>
  <PackageReference Include="coverlet.msbuild" Version="3.2.0">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  </PackageReference>
</ItemGroup>
```

### Ejecución

```bash
# Ejecutar pruebas con cobertura
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover

# Con thresholds de calidad
dotnet test /p:CollectCoverage=true /p:Threshold=80 /p:ThresholdType=line
```

### Configuración avanzada

```xml
<!-- MiProyecto.Tests.csproj -->
<PropertyGroup>
  <CollectCoverage>true</CollectCoverage>
  <CoverletOutput>./coverage/</CoverletOutput>
  <CoverletOutputFormat>opencover,json,lcov</CoverletOutputFormat>
  <Exclude>[*.Tests]*,[*]*.Designer</Exclude>
</PropertyGroup>
```

### Limitaciones en .NET 4.8

- Requiere formato de proyecto SDK-style (no siempre compatible)
- Funcionalidad reducida comparada con .NET Core
- Mejor opción: OpenCover para proyectos legacy

## 5. Análisis de calidad del código con SonarQube

SonarQube es una plataforma de análisis continuo de calidad de código que identifica bugs, vulnerabilidades y code smells.

### Componentes

**SonarQube Server**: Servidor central que almacena y muestra resultados.

**SonarScanner**: Cliente que analiza código y envía resultados al servidor.

**SonarLint**: Plugin IDE para análisis en tiempo real.

### Instalación local

```bash
# Descargar SonarQube Community Edition
# Ejecutar servidor
.\bin\windows-x86-64\StartSonar.bat

# Acceder a http://localhost:9000 (admin/admin)
```

### Configuración para .NET 4.8

```bash
# Instalar SonarScanner para .NET Framework
dotnet tool install --global dotnet-sonarscanner

# Iniciar análisis
SonarScanner.MSBuild.exe begin ^
  /k:"MiProyecto" ^
  /d:sonar.host.url="http://localhost:9000" ^
  /d:sonar.login="admin-token" ^
  /d:sonar.cs.opencover.reportsPaths="coverage.xml"

# Compilar proyecto
msbuild MiSolucion.sln /t:Rebuild

# Finalizar análisis y enviar
SonarScanner.MSBuild.exe end /d:sonar.login="admin-token"
```

### Métricas principales

**Bugs**: Errores que afectan funcionalidad.

**Vulnerabilidades**: Problemas de seguridad.

**Code Smells**: Problemas de mantenibilidad.

**Deuda técnica**: Tiempo estimado para resolver issues.

**Duplicación**: Porcentaje de código duplicado.

### Ejemplo de reglas detectadas

```csharp
// Code Smell: Método demasiado complejo (Cognitive Complexity > 15)
public string ProcesarPedido(Pedido pedido)
{
    if (pedido != null)
    {
        if (pedido.Cliente != null)
        {
            if (pedido.Items.Count > 0)
            {
                foreach (var item in pedido.Items)
                {
                    if (item.Stock > 0)
                    {
                        // ...más anidamiento
                    }
                }
            }
        }
    }
}

// Vulnerabilidad: SQL Injection
string query = "SELECT * FROM Users WHERE Id = " + userId; // Usar parámetros

// Bug: Posible NullReferenceException
var nombre = cliente.Nombre.ToUpper(); // Validar antes
```

### Quality Gates

```json
// Configuración de Quality Gate personalizado
{
  "conditions": [
    { "metric": "new_coverage", "op": "LT", "error": "80" },
    { "metric": "new_bugs", "op": "GT", "error": "0" },
    { "metric": "new_vulnerabilities", "op": "GT", "error": "0" },
    { "metric": "new_duplicated_lines_density", "op": "GT", "error": "3" }
  ]
}
```

## 6. Code Metrics en Visual Studio

Visual Studio incluye herramientas nativas para calcular métricas de código que evalúan mantenibilidad y complejidad.

### Métricas disponibles

**Maintainability Index (0-100)**: Facilidad de mantenimiento. >20 aceptable, >80 excelente.

**Cyclomatic Complexity**: Número de caminos de ejecución. <10 simple, >20 complejo.

**Depth of Inheritance**: Niveles de herencia. <5 recomendado.

**Class Coupling**: Dependencias entre clases. <9 bajo acoplamiento.

**Lines of Code**: Líneas de código fuente y ejecutable.

### Cálculo de métricas

```
Menú: Analyze → Calculate Code Metrics → For Solution
```

### Interpretación de resultados

```csharp
// Ejemplo con baja mantenibilidad (Index: 45)
public class ServicioPedidos
{
    private IRepositorio repo;
    private ILogger logger;
    private IEmail email;
    private IPago pago;
    private IInventario inventario;
    // 8 dependencias más...
    
    public void ProcesarPedido(Pedido p)
    {
        // 150 líneas de código
        // Cyclomatic Complexity: 28
        // Depth of Inheritance: 3
        // Class Coupling: 15
    }
}

// Refactorizado con mejor mantenibilidad (Index: 78)
public class ServicioPedidos
{
    private readonly IPedidoOrchestrator _orchestrator;
    
    public ServicioPedidos(IPedidoOrchestrator orchestrator)
    {
        _orchestrator = orchestrator;
    }
    
    public void ProcesarPedido(Pedido pedido)
    {
        _orchestrator.Procesar(pedido);
    }
}
```

### Exportación de resultados

```xml
<!-- Resultados exportados en XML -->
<CodeMetricsReport>
  <Targets>
    <Target Name="MiProyecto.dll">
      <Modules>
        <Module Name="MiProyecto">
          <Metrics>
            <Metric Name="MaintainabilityIndex" Value="72" />
            <Metric Name="CyclomaticComplexity" Value="45" />
            <Metric Name="ClassCoupling" Value="12" />
            <Metric Name="DepthOfInheritance" Value="2" />
            <Metric Name="LinesOfCode" Value="850" />
          </Metrics>
        </Module>
      </Modules>
    </Target>
  </Targets>
</CodeMetricsReport>
```

## 7. Visual Studio Code Analysis

Code Analysis (FxCop) analiza ensamblados compilados para detectar violaciones de reglas de diseño, nomenclatura, rendimiento y seguridad.

### Habilitación

```xml
<!-- En archivo .csproj -->
<PropertyGroup>
  <CodeAnalysisRuleSet>AllRules.ruleset</CodeAnalysisRuleSet>
  <RunCodeAnalysis>true</RunCodeAnalysis>
  <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
</PropertyGroup>
```

### Categorías de reglas

**Design**: Patrones SOLID, uso correcto de interfaces.

**Globalization**: Problemas de internacionalización.

**Naming**: Convenciones de nomenclatura.

**Performance**: Optimizaciones de rendimiento.

**Security**: Vulnerabilidades de seguridad.

**Usage**: Uso correcto de APIs .NET.

### Ejemplo de violaciones

```csharp
// CA1031: No capturar tipos de excepción generales
try
{
    ProcesarDatos();
}
catch (Exception ex)  // Violación: Demasiado genérico
{
    Log(ex);
}

// Correcto: Capturar excepciones específicas
try
{
    ProcesarDatos();
}
catch (InvalidOperationException ex)
{
    Log(ex);
}
catch (ArgumentNullException ex)
{
    Log(ex);
}

// CA1806: No ignorar resultados de métodos
cliente.Nombre.Trim();  // Violación: Resultado no utilizado

// Correcto
var nombreLimpio = cliente.Nombre.Trim();

// CA2000: Liberar objetos antes de perder el ámbito
var conexion = new SqlConnection(connectionString);  // Violación
conexion.Open();

// Correcto
using (var conexion = new SqlConnection(connectionString))
{
    conexion.Open();
    // usar conexión
}
```

### Configuración personalizada

```xml
<!-- CustomRules.ruleset -->
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="Reglas Personalizadas" ToolsVersion="16.0">
  <Rules AnalyzerId="Microsoft.Analyzers.ManagedCodeAnalysis" RuleNamespace="Microsoft.Rules.Managed">
    <Rule Id="CA1001" Action="Error" />
    <Rule Id="CA1031" Action="Warning" />
    <Rule Id="CA1062" Action="Warning" />
    <Rule Id="CA2000" Action="Error" />
  </Rules>
</RuleSet>
```

### Supresión de advertencias

```csharp
using System.Diagnostics.CodeAnalysis;

[SuppressMessage("Microsoft.Design", "CA1031:DoNotCatchGeneralExceptionTypes",
    Justification = "Necesario para logging centralizado")]
public void ProcesarLote()
{
    try
    {
        // procesamiento
    }
    catch (Exception ex)
    {
        LogCentral.Error(ex);
    }
}
```

## 8. Comparativa de herramientas

### Tabla comparativa

| Característica | OpenCover | Coverlet | VS Coverage | SonarQube | Code Metrics | Code Analysis |
|----------------|-----------|----------|-------------|-----------|--------------|---------------|
| **Tipo** | Cobertura | Cobertura | Cobertura | Calidad/Cobertura | Complejidad | Análisis estático |
| **Compatibilidad .NET 4.8** | Excelente | Limitada | Excelente | Excelente | Excelente | Excelente |
| **Costo** | Gratis | Gratis | VS Pro+ | Gratis (Community) | Incluido | Incluido |
| **Configuración** | Compleja | Media | Simple | Compleja | Simple | Simple |
| **Reportes visuales** | Sí (con ReportGenerator) | Sí | Sí | Excelente | Básico | Lista de issues |
| **CI/CD** | Excelente | Excelente | Limitado | Excelente | Manual | Automático |
| **Detección bugs** | No | No | No | Sí | No | Sí |
| **Métricas de calidad** | No | No | No | Sí | Sí | Limitado |
| **Velocidad** | Media | Rápida | Rápida | Lenta | Rápida | Media |

### Recomendaciones de uso

**Para proyectos pequeños/medianos**: OpenCover + ReportGenerator + Code Analysis integrado

**Para equipos sin licencias VS Enterprise**: OpenCover + SonarQube Community

**Para empresas grandes**: Visual Studio Coverage + SonarQube Enterprise + Quality Gates

**Para CI/CD en .NET 4.8**: OpenCover + ReportGenerator + integración Azure DevOps/Jenkins

## 9. Integración en flujo de trabajo

### Pipeline típico

```yaml
# Ejemplo Azure DevOps Pipeline
stages:
- stage: Build
  jobs:
  - job: BuildAndTest
    steps:
    - task: NuGetRestore@1
    
    - task: MSBuild@1
      inputs:
        solution: '**/*.sln'
        configuration: 'Release'
    
    - script: |
        OpenCover.Console.exe ^
          -target:"vstest.console.exe" ^
          -targetargs:"**\*.Tests.dll" ^
          -filter:"+[MiApp*]* -[*.Tests]*" ^
          -output:coverage.xml
      displayName: 'Ejecutar pruebas con cobertura'
    
    - script: |
        ReportGenerator.exe ^
          -reports:coverage.xml ^
          -targetdir:coverage-report ^
          -reporttypes:HtmlInline_AzurePipelines;Cobertura
      displayName: 'Generar reporte de cobertura'
    
    - task: PublishCodeCoverageResults@1
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '**/coverage.xml'
        reportDirectory: '**/coverage-report'
    
    - task: SonarQubeAnalyze@5
      displayName: 'Análisis SonarQube'
    
    - task: SonarQubePublish@5
      inputs:
        pollingTimeoutSec: '300'
```

### Thresholds de calidad

```csharp
// Configurar thresholds en pruebas
[TestClass]
public class CoberturaTests
{
    [TestMethod]
    public void ValidarCoberturaMinimaLineas()
    {
        // Leer resultado de cobertura
        var cobertura = ObtenerCobertura("coverage.xml");
        Assert.IsTrue(cobertura.LineRate >= 0.80, 
            $"Cobertura de líneas: {cobertura.LineRate:P}. Mínimo requerido: 80%");
    }
}
```

### Buenas prácticas

1. **Establecer mínimos realistas**: 70-80% para código nuevo, 60% para legacy
2. **Excluir código generado**: Modelos de EF, Designer files, etc.
3. **Priorizar cobertura de branches**: Más valiosa que cobertura de líneas
4. **Revisar métricas regularmente**: En cada PR y sprint review
5. **No obsesionarse con 100%**: Foco en código crítico de negocio

## 10. Ejercicio práctico

### Escenario

Analizar y mejorar la cobertura y métricas de un servicio de facturación.

### Código inicial

```csharp
public class ServicioFacturacion
{
    private RepositorioFacturas repo;
    
    public decimal CalcularTotal(Factura factura)
    {
        decimal total = 0;
        
        if (factura != null && factura.Lineas != null)
        {
            foreach (var linea in factura.Lineas)
            {
                if (linea.Cantidad > 0 && linea.PrecioUnitario > 0)
                {
                    var subtotal = linea.Cantidad * linea.PrecioUnitario;
                    
                    if (linea.Descuento > 0)
                    {
                        subtotal -= subtotal * (linea.Descuento / 100);
                    }
                    
                    total += subtotal;
                }
            }
            
            if (factura.Cliente.EsPremium && total > 1000)
            {
                total *= 0.95m;
            }
        }
        
        return total;
    }
}
```

### Pruebas con cobertura completa

```csharp
[TestClass]
public class ServicioFacturacionTests
{
    private ServicioFacturacion servicio;
    
    [TestInitialize]
    public void Setup()
    {
        servicio = new ServicioFacturacion();
    }
    
    [TestMethod]
    public void CalcularTotal_FacturaNull_RetornaCero()
    {
        var resultado = servicio.CalcularTotal(null);
        Assert.AreEqual(0, resultado);
    }
    
    [TestMethod]
    public void CalcularTotal_LineasNull_RetornaCero()
    {
        var factura = new Factura { Lineas = null };
        var resultado = servicio.CalcularTotal(factura);
        Assert.AreEqual(0, resultado);
    }
    
    [TestMethod]
    public void CalcularTotal_SinDescuento_RetornaSumaSimple()
    {
        var factura = CrearFactura(cantidad: 10, precio: 50, descuento: 0);
        var resultado = servicio.CalcularTotal(factura);
        Assert.AreEqual(500, resultado);
    }
    
    [TestMethod]
    public void CalcularTotal_ConDescuento_AplicaDescuento()
    {
        var factura = CrearFactura(cantidad: 10, precio: 100, descuento: 10);
        var resultado = servicio.CalcularTotal(factura);
        Assert.AreEqual(900, resultado); // 1000 - 10%
    }
    
    [TestMethod]
    public void CalcularTotal_ClientePremiumMayor1000_AplicaDescuentoAdicional()
    {
        var factura = CrearFactura(cantidad: 20, precio: 60, descuento: 0);
        factura.Cliente.EsPremium = true;
        var resultado = servicio.CalcularTotal(factura);
        Assert.AreEqual(1140, resultado); // 1200 * 0.95
    }
    
    private Factura CrearFactura(int cantidad, decimal precio, decimal descuento)
    {
        return new Factura
        {
            Cliente = new Cliente { EsPremium = false },
            Lineas = new List<LineaFactura>
            {
                new LineaFactura
                {
                    Cantidad = cantidad,
                    PrecioUnitario = precio,
                    Descuento = descuento
                }
            }
        };
    }
}

// Resultado: Cobertura 100%, Cyclomatic Complexity: 8
```

### Análisis de resultados

```bash
# Ejecutar OpenCover
OpenCover.Console.exe -target:vstest.console.exe ^
  -targetargs:"bin\Debug\Facturacion.Tests.dll" ^
  -filter:"+[Facturacion]* -[*.Tests]*" ^
  -output:coverage.xml

# Generar reporte
ReportGenerator.exe -reports:coverage.xml -targetdir:report

# Resultados esperados:
# Line Coverage: 100%
# Branch Coverage: 100%
# Cyclomatic Complexity: 8 (Aceptable)
# Maintainability Index: 65 (Mejorable)
```

## Conclusión

Las herramientas de cobertura y métricas son esenciales para mantener calidad en proyectos .NET 4.8. La combinación de OpenCover para cobertura, SonarQube para análisis de calidad, y las herramientas nativas de Visual Studio proporciona una visión completa del estado del código. Establecer thresholds realistas y automatizar análisis en CI/CD garantiza sostenibilidad a largo plazo.

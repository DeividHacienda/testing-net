# Buenas prácticas en CI/CD y automatización de pruebas

## Introducción

La integración continua y el despliegue continuo (CI/CD) son prácticas fundamentales para garantizar la calidad del software. En .NET 4.8, implementar una estrategia sólida de automatización de pruebas permite detectar errores tempranamente y mantener un código estable.

## Estrategias de ejecución de pruebas en pipelines

### Organización por tipo de prueba

Las pruebas deben ejecutarse en diferentes etapas del pipeline según su naturaleza:

**1. Build Stage (Compilación)**
- Análisis estático del código (Roslyn Analyzers, FxCop)
- Verificación de estándares de código (StyleCop)

**2. Test Stage (Pruebas rápidas)**
- Pruebas unitarias (< 1 segundo por test)
- Cobertura de código básica

**3. Integration Stage**
- Pruebas de integración con bases de datos
- Pruebas de servicios externos (APIs, WCF)

**4. UI Stage**
- Pruebas de interfaz con Selenium
- Pruebas end-to-end críticas

### Ejemplo de configuración Azure DevOps

```yaml
stages:
- stage: Build
  jobs:
  - job: Compile
    steps:
    - task: NuGetRestore@1
    - task: VSBuild@1
      inputs:
        solution: '**/*.sln'
        configuration: 'Release'
    - task: RunCodeAnalysis@1

- stage: UnitTests
  jobs:
  - job: RunUnitTests
    steps:
    - task: VSTest@2
      inputs:
        testSelector: 'testAssemblies'
        testAssemblyVer2: |
          **\*UnitTests.dll
          !**\obj\**
        runInParallel: true

- stage: IntegrationTests
  jobs:
  - job: RunIntegrationTests
    steps:
    - task: VSTest@2
      inputs:
        testSelector: 'testAssemblies'
        testAssemblyVer2: |
          **\*IntegrationTests.dll
```

## Cuándo ejecutar cada tipo de prueba

| Tipo de prueba | Frecuencia | Momento |
|----------------|------------|---------|
| Unitarias | Cada commit | Pre-commit hook / Build |
| Integración | Cada PR | Antes de merge |
| UI | Nightly / Semanal | Deploy a staging |
| Regresión completa | Release | Antes de producción |

### Pre-commit hooks

```bash
# Ejemplo de pre-commit hook para .NET 4.8
#!/bin/sh
echo "Ejecutando pruebas unitarias..."
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\MSTest.exe" /testcontainer:Tests\bin\Debug\MyApp.UnitTests.dll

if [ $? -ne 0 ]; then
    echo "Las pruebas unitarias fallaron. Commit cancelado."
    exit 1
fi
```

## Métricas y Quality Gates: ¿Qué se mide?

### 1. Cobertura de código

**Métrica**: Porcentaje de líneas de código ejecutadas durante las pruebas.

**Herramienta**: Coverlet, OpenCover, dotCover

**Umbrales recomendados**:
- Cobertura mínima global: **70%**
- Código crítico (servicios, lógica de negocio): **85%**
- Código nuevo: **80%**

```xml
<!-- Ejemplo configuración Coverlet en .csproj -->
<PropertyGroup>
  <CoverletOutputFormat>cobertura</CoverletOutputFormat>
  <CoverletThreshold>70</CoverletThreshold>
  <CoverletThresholdType>line,branch,method</CoverletThresholdType>
</PropertyGroup>
```

**Bloqueo de commit**: Se rechaza si la cobertura cae por debajo del umbral establecido.

### 2. Tasa de éxito de pruebas

**Métrica**: Porcentaje de pruebas que pasan exitosamente.

**Umbral**: **100%** de las pruebas deben pasar.

**Bloqueo de commit**: Cualquier prueba fallida bloquea el commit/merge.

```csharp
// Ejemplo: configuración en Azure DevOps
// Si alguna prueba falla, el build se marca como fallido
```

### 3. Calidad del código (SonarQube)

**Métricas medidas**:
- **Bugs**: Defectos que causan errores en runtime
- **Vulnerabilidades**: Problemas de seguridad
- **Code Smells**: Código difícil de mantener
- **Deuda técnica**: Tiempo estimado para arreglar problemas
- **Duplicación de código**: Porcentaje de código duplicado

**Umbrales recomendados**:
- Bugs nuevos: **0**
- Vulnerabilidades nuevas: **0**
- Code Smells críticos: **0**
- Cobertura en código nuevo: **>80%**
- Duplicación: **<3%**

**Bloqueo de commit**: Se rechaza si hay bugs o vulnerabilidades nuevas.

```json
// Ejemplo quality gate en SonarQube
{
  "conditions": [
    {
      "metric": "new_bugs",
      "op": "GT",
      "error": "0"
    },
    {
      "metric": "new_vulnerabilities",
      "op": "GT",
      "error": "0"
    },
    {
      "metric": "new_coverage",
      "op": "LT",
      "error": "80"
    }
  ]
}
```

### 4. Complejidad ciclomática

**Métrica**: Número de caminos independientes a través del código.

**Umbral**: Máximo **10** por método (recomendado), crítico si supera **15**.

**Bloqueo de commit**: Se rechaza si se introduce un método con complejidad >15.

```csharp
// Ejemplo de código con alta complejidad (evitar)
public decimal CalcularDescuento(Cliente cliente, Producto producto, DateTime fecha)
{
    if (cliente.EsPremium)
    {
        if (producto.Categoria == "Electronica")
        {
            if (fecha.DayOfWeek == DayOfWeek.Friday)
            {
                // Complejidad aumenta con cada if anidado
                return 0.25m;
            }
        }
    }
    // ... más condiciones
}

// Mejor: refactorizar usando estrategia
public decimal CalcularDescuento(Cliente cliente, Producto producto, DateTime fecha)
{
    var estrategia = _descuentoFactory.ObtenerEstrategia(cliente, producto, fecha);
    return estrategia.Calcular();
}
```

### 5. Tiempo de ejecución de pruebas

**Métrica**: Duración total del pipeline.

**Objetivos**:
- Pruebas unitarias: **<5 minutos**
- Pruebas de integración: **<15 minutos**
- Pipeline completo: **<30 minutos**

**Acción**: Optimizar o paralelizar si se exceden los tiempos.

### 6. Pruebas flaky

**Métrica**: Número de pruebas que fallan intermitentemente.

**Umbral**: **0 pruebas flaky** permitidas en el build principal.

**Acción**: Identificar, arreglar o deshabilitar temporalmente con documentación del issue.

```csharp
[TestMethod]
[Ignore("Flaky test - Issue #1234: falla intermitentemente por timeout")]
public void TestConProblemasDeTimeout()
{
    // Este test necesita refactorización
}
```

## Criterios de rechazo de commits

Un commit/pull request será **rechazado automáticamente** si:

### ❌ Bloqueos críticos

1. **Fallan pruebas unitarias**: Cualquier prueba unitaria fallida bloquea el commit.
   
2. **Cobertura insuficiente**: El código nuevo tiene menos del 80% de cobertura.

3. **Bugs nuevos**: SonarQube detecta nuevos bugs en el código.

4. **Vulnerabilidades de seguridad**: Se introducen vulnerabilidades críticas o altas.

5. **Build fallido**: El código no compila o hay errores de compilación.

6. **Análisis estático fallido**: Violación de reglas críticas (nullref potenciales, disposición incorrecta de recursos).

### ⚠️ Advertencias (revisión manual requerida)

1. **Code Smells mayores**: Más de 5 code smells en el código nuevo.

2. **Complejidad elevada**: Métodos con complejidad ciclomática entre 10-15.

3. **Duplicación de código**: Más del 3% de código duplicado.

4. **Cobertura reducida**: La cobertura global cae más del 2%.

### Ejemplo de configuración en Azure DevOps

```yaml
# azure-pipelines.yml
trigger:
- main
- develop

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildConfiguration: 'Release'
  minimumCoverage: 70
  minimumNewCoverage: 80

steps:
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'

- task: VSBuild@1
  inputs:
    solution: '$(solution)'
    configuration: '$(buildConfiguration)'

- task: VSTest@2
  inputs:
    testSelector: 'testAssemblies'
    testAssemblyVer2: '**\*Tests.dll'
    runInParallel: true
    codeCoverageEnabled: true

- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(System.DefaultWorkingDirectory)/**/coverage.cobertura.xml'
    failIfCoverageEmpty: true

- task: SonarQubeAnalyze@5
  displayName: 'Análisis SonarQube'

- task: SonarQubePublish@5
  inputs:
    pollingTimeoutSec: '300'

# Quality Gate Check
- powershell: |
    $coverage = [xml](Get-Content coverage.cobertura.xml)
    $lineRate = [double]$coverage.coverage.'line-rate' * 100
    
    if ($lineRate -lt $(minimumCoverage)) {
      Write-Error "Cobertura insuficiente: $lineRate% (mínimo: $(minimumCoverage)%)"
      exit 1
    }
  displayName: 'Verificar cobertura mínima'
```

## Paralelización y optimización

### Ejecución paralela de pruebas

```csharp
// MSTest: habilitar paralelización
[assembly: Parallelize(Workers = 4, Scope = ExecutionScope.ClassLevel)]

// xUnit: configuración en xunit.runner.json
{
  "parallelizeTestCollections": true,
  "maxParallelThreads": 4
}
```

### Optimización de tiempos

**Técnicas**:
1. **Caché de dependencias**: Almacenar paquetes NuGet en caché.
2. **Build incremental**: Solo compilar proyectos modificados.
3. **Pruebas selectivas**: Ejecutar solo pruebas afectadas por cambios.
4. **Artifacts compartidos**: Reutilizar binarios entre stages.

```yaml
# Ejemplo de caché en Azure DevOps
- task: Cache@2
  inputs:
    key: 'nuget | "$(Agent.OS)" | **/packages.lock.json'
    path: $(NUGET_PACKAGES)
  displayName: 'Cache NuGet packages'
```

## Gestión de pruebas flaky

### Identificación

```powershell
# Script para detectar pruebas flaky (ejecutar 10 veces cada test)
$testDll = "Tests\bin\Debug\MyApp.Tests.dll"
$results = @()

for ($i = 1; $i -le 10; $i++) {
    $result = & vstest.console.exe $testDll /Logger:trx
    $results += $result
}

# Analizar resultados para encontrar inconsistencias
```

### Estrategias de corrección

1. **Eliminar dependencias de tiempo**: No usar `Thread.Sleep()`, usar mocks con timeout configurables.

```csharp
// ❌ Evitar
[TestMethod]
public void TestConSleep()
{
    var servicio = new MiServicio();
    servicio.IniciarProceso();
    Thread.Sleep(5000); // Flaky: depende del timing
    Assert.IsTrue(servicio.ProcesoCompletado);
}

// ✅ Mejor
[TestMethod]
public void TestConEsperaCondicional()
{
    var servicio = new MiServicio();
    servicio.IniciarProceso();
    
    var esperado = SpinWait.SpinUntil(() => servicio.ProcesoCompletado, TimeSpan.FromSeconds(5));
    Assert.IsTrue(esperado);
}
```

2. **Aislar estado compartido**: Cada test debe ser independiente.

```csharp
[TestClass]
public class ServicioTests
{
    private MiServicio _servicio;
    
    [TestInitialize]
    public void Setup()
    {
        // Crear nueva instancia para cada test
        _servicio = new MiServicio();
    }
    
    [TestCleanup]
    public void Cleanup()
    {
        // Limpiar recursos
        _servicio?.Dispose();
    }
}
```

3. **Controlar aleatoriedad**: Usar seeds fijas en entorno de pruebas.

```csharp
// En producción
var random = new Random();

// En tests
var random = new Random(42); // Seed fija = resultados reproducibles
```

## Mejores prácticas en informes y logs

### Estructura de informes

1. **Resumen ejecutivo**: Métricas clave al inicio.
2. **Desglose por tipo**: Unitarias, integración, UI.
3. **Fallos detallados**: Stack trace completo, contexto.
4. **Tendencias históricas**: Evolución de métricas en el tiempo.

### Logs efectivos en tests

```csharp
[TestMethod]
public void TestConLogsDetallados()
{
    // Arrange
    Console.WriteLine("[ARRANGE] Configurando servicio con timeout de 30s");
    var config = new ConfiguracionServicio { Timeout = 30 };
    var servicio = new MiServicio(config);
    
    // Act
    Console.WriteLine("[ACT] Ejecutando llamada al servicio");
    var resultado = servicio.Ejecutar();
    
    // Assert
    Console.WriteLine($"[ASSERT] Resultado obtenido: {resultado}");
    Assert.IsNotNull(resultado);
}
```

### Integración con herramientas de reporte

```xml
<!-- Configuración de ReportGenerator -->
<PropertyGroup>
  <ReportGeneratorReportTypes>
    Html;Badges;Cobertura;SonarQube
  </ReportGeneratorReportTypes>
</PropertyGroup>
```

## Estrategias de rollback

### Criterios de rollback automático

Un despliegue se revierte automáticamente si:

1. **Fallos en smoke tests**: Las pruebas básicas post-deploy fallan.
2. **Aumento de errores**: +20% de errores en producción en 10 minutos.
3. **Degradación de performance**: Tiempo de respuesta >50% más lento.
4. **Health checks fallidos**: Los endpoints de salud no responden.

### Implementación de rollback

```yaml
# Azure DevOps: deployment con rollback
- deployment: DeployProduction
  environment: 'production'
  strategy:
    runOnce:
      deploy:
        steps:
        - task: AzureWebApp@1
          inputs:
            appName: 'myapp-prod'
            package: '$(Pipeline.Workspace)/drop/*.zip'
        
        - task: VSTest@2
          displayName: 'Smoke Tests'
          inputs:
            testSelector: 'testAssemblies'
            testAssemblyVer2: '**\*SmokeTests.dll'
          continueOnError: false
        
      on:
        failure:
          steps:
          - task: AzureAppServiceManage@0
            inputs:
              azureSubscription: 'MySubscription'
              Action: 'Swap Slots'
              WebAppName: 'myapp-prod'
              SourceSlot: 'production'
              SwapWithSlot: 'previous'
```

## Políticas de calidad (Quality Gates)

### Configuración recomendada

**Nivel 1 - Merge a develop**:
- Todas las pruebas unitarias pasan
- Cobertura código nuevo >80%
- 0 bugs nuevos
- 0 vulnerabilidades

**Nivel 2 - Merge a main**:
- Todo lo de Nivel 1
- Pruebas de integración exitosas
- Revisión de código aprobada por 2 personas
- Cobertura global >70%

**Nivel 3 - Deploy a producción**:
- Todo lo de Nivel 2
- Pruebas de regresión completas
- Pruebas de performance dentro de umbrales
- Aprobación manual de product owner

### Implementación en Azure DevOps

```yaml
# Branch policies para main
resources:
  repositories:
  - repository: self
    type: git
    ref: refs/heads/main

pr:
  branches:
    include:
    - main
  paths:
    exclude:
    - docs/*
    - README.md

variables:
  - name: 'qualityGatesPassed'
    value: false

stages:
- stage: QualityGates
  jobs:
  - job: CheckQuality
    steps:
    - task: SonarQubePublish@5
      inputs:
        pollingTimeoutSec: '300'
    
    - powershell: |
        # Verificar que todos los quality gates pasaron
        $sonarStatus = Get-Content 'sonar-report.json' | ConvertFrom-Json
        
        if ($sonarStatus.qualityGateStatus -ne 'OK') {
          Write-Error "Quality Gate failed"
          exit 1
        }
        
        Write-Host "##vso[task.setvariable variable=qualityGatesPassed]true"
```

## Resumen de umbrales y criterios de bloqueo

| Métrica | Umbral mínimo | Bloquea commit | Herramienta |
|---------|---------------|----------------|-------------|
| Cobertura global | 70% | ✅ | Coverlet |
| Cobertura código nuevo | 80% | ✅ | Coverlet |
| Pruebas unitarias | 100% pasan | ✅ | MSTest/xUnit/NUnit |
| Bugs nuevos | 0 | ✅ | SonarQube |
| Vulnerabilidades | 0 críticas/altas | ✅ | SonarQube |
| Complejidad ciclomática | <15 por método | ✅ | Code Metrics |
| Code Smells críticos | 0 | ⚠️ | SonarQube |
| Duplicación | <3% | ⚠️ | SonarQube |
| Tiempo pruebas unitarias | <5 min | ⚠️ | - |
| Pruebas flaky | 0 | ⚠️ | Análisis histórico |

**Leyenda**:
- ✅ = Bloqueo automático del commit/merge
- ⚠️ = Advertencia, requiere revisión manual

## Checklist final para implementación

- [ ] Configurar análisis estático en el build
- [ ] Definir umbrales de cobertura (global y nuevo código)
- [ ] Integrar SonarQube con quality gates
- [ ] Implementar ejecución paralela de pruebas
- [ ] Configurar caché de dependencias
- [ ] Establecer políticas de branch protection
- [ ] Definir criterios de rollback automático
- [ ] Configurar notificaciones de fallos
- [ ] Documentar proceso de resolución de pruebas flaky
- [ ] Entrenar al equipo en las políticas establecidas

## Conclusión

Implementar estas prácticas en CI/CD para .NET 4.8 garantiza un pipeline robusto que detecta problemas tempranamente y mantiene la calidad del código. Los quality gates actúan como guardianes automáticos, bloqueando código de baja calidad antes de que llegue a producción.

La clave del éxito está en la **automatización inteligente**: medir lo correcto, establecer umbrales realistas pero exigentes, y hacer que el pipeline sea un aliado del desarrollador, no un obstáculo.
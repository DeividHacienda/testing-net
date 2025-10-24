# Flujo completo: Desarrollo → Staging → Producción

## Introducción

Este documento explica de forma clara y práctica el flujo completo desde que un desarrollador escribe código hasta que llega a producción en .NET 4.8.

## Arquitectura de entornos

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ DESARROLLO  │───▶│   STAGING   │───▶│ PRODUCCIÓN  │
│   (DEV)     │    │    (STG)    │    │   (PROD)    │
└─────────────┘    └─────────────┘    └─────────────┘
     Local              UAT            Cliente final
```

### Características de cada entorno

| Aspecto | Desarrollo | Staging | Producción |
|---------|-----------|---------|------------|
| **Propósito** | Desarrollo y pruebas unitarias | QA y pruebas de integración | Usuarios finales |
| **Datos** | Datos sintéticos/mock | Copia anonimizada de producción | Datos reales |
| **Acceso** | Todos los desarrolladores | QA, PM, stakeholders | Solo usuarios finales |
| **Uptime** | No crítico | 95% | 99.9% |
| **Monitoreo** | Básico | Completo | Completo + alertas 24/7 |
| **Rollback** | No necesario | Manual | Automático |

## FASE 1: Desarrollo Local (Rama Feature)

### 1. El desarrollador crea su rama

```bash
git checkout develop
git pull origin develop
git checkout -b feature/JIRA-1234-nueva-funcionalidad
```

### 2. Desarrolla y prueba localmente

**Mientras desarrolla:**
- Escribe código siguiendo estándares del equipo
- Crea pruebas unitarias para su código
- Ejecuta las pruebas localmente: `dotnet test`
- Compila sin errores ni warnings

**Ejemplo de desarrollo:**
```csharp
// 1. Implementa la funcionalidad
public class ReporteService
{
    public byte[] GenerarReportePDF(int clienteId)
    {
        // Implementación...
    }
}

// 2. Crea las pruebas unitarias
[TestClass]
public class ReporteServiceTests
{
    [TestMethod]
    public void GenerarReportePDF_ClienteValido_RetornaPDF()
    {
        // Arrange
        var service = new ReporteService();
        
        // Act
        var resultado = service.GenerarReportePDF(123);
        
        // Assert
        Assert.IsNotNull(resultado);
        Assert.IsTrue(resultado.Length > 0);
    }
}
```

### 3. Pre-commit: Validaciones antes de hacer commit

**Antes de hacer commit, el desarrollador debe:**

✅ Compilar el proyecto completo sin errores
```bash
msbuild MySolution.sln /p:Configuration=Release
```

✅ Ejecutar todas las pruebas unitarias
```bash
vstest.console.exe Tests\bin\Release\*.UnitTests.dll
```

✅ Verificar cobertura de código (opcional localmente)
```bash
dotnet test /p:CollectCoverage=true
```

**Si todo está bien → Hacer commit**

### 4. Commit y push

```bash
git add .
git commit -m "feat: implementar generación de reportes en PDF"
git push origin feature/JIRA-1234-nueva-funcionalidad
```

## FASE 2: Pull Request (PR) a Develop

### 1. Crear el Pull Request

El desarrollador crea un PR en Azure DevOps/GitHub con:

**Información requerida:**
- **Título**: "JIRA-1234: Generación de reportes en PDF"
- **Descripción**: Qué hace, por qué, cómo probarlo
- **Link al ticket**: JIRA-1234
- **Screenshots**: Si hay cambios visuales

**Ejemplo de descripción:**
```markdown
## Descripción
Implementa la generación de reportes de clientes en formato PDF usando iTextSharp.

## Cambios realizados
- Nuevo servicio `ReporteService`
- Endpoint `/api/reportes/clientes/{id}`
- 15 pruebas unitarias (cobertura 92%)

## Cómo probar
1. Llamar a GET /api/reportes/clientes/123
2. Descargar el PDF generado
3. Verificar que contiene datos del cliente
```

### 2. Pipeline automático de validación del PR

**Cuando se crea el PR, automáticamente se ejecuta:**

#### Paso 1: Compilación (2-3 minutos)
- Restaurar paquetes NuGet
- Compilar la solución completa
- Verificar que no hay errores ni warnings

**❌ Si falla la compilación → PR bloqueado, no se puede continuar**

#### Paso 2: Pruebas unitarias (3-5 minutos)
- Ejecutar todas las pruebas unitarias del proyecto
- Generar reporte de cobertura

**❌ Si alguna prueba falla → PR bloqueado**

#### Paso 3: Análisis de calidad (3-5 minutos)
- **SonarQube** analiza el código buscando:
  - Bugs (errores potenciales)
  - Vulnerabilidades de seguridad
  - Code smells (mal código)
  - Código duplicado
  - Complejidad ciclomática

**❌ Si hay bugs nuevos o vulnerabilidades → PR bloqueado**

#### Paso 4: Verificación de cobertura (1 minuto)
- Calcula el % de código cubierto por pruebas
- Verifica umbrales mínimos

**❌ Si la cobertura es < 70% global o < 80% en código nuevo → PR bloqueado**

### 3. Revisión de código (Code Review)

**Mínimo 2 desarrolladores seniors deben revisar y aprobar el código:**

**Aspectos que revisan:**
- ¿El código sigue los estándares del equipo?
- ¿Es legible y mantenible?
- ¿Las pruebas son correctas y suficientes?
- ¿Hay casos edge no contemplados?
- ¿La documentación es clara?

**Ejemplo de feedback:**
```csharp
// ❌ Comentario del revisor
public DataTable GetClientes()  
{
    // Problema 1: No usar DataTable, usar DTOs
    // Problema 2: No concatenar SQL, riesgo de SQL injection
    return _db.ExecuteQuery("SELECT * FROM Clientes WHERE Id = " + id);
}

// ✅ Sugerencia del revisor
public async Task<ClienteDto> ObtenerClientePorIdAsync(int id)
{
    return await _clienteRepository.ObtenerPorIdAsync(id);
}
```

### 4. Criterios para aprobar el PR

El PR solo se puede mergear si:

✅ **Automático:**
- Build exitoso (compila sin errores)
- Todas las pruebas unitarias pasan (100%)
- Cobertura ≥ 70% global y ≥ 80% en código nuevo
- 0 bugs nuevos (SonarQube)
- 0 vulnerabilidades críticas/altas
- Complejidad ciclomática < 15 por método

✅ **Manual:**
- 2 aprobaciones de code review
- Arquitecto aprueba (si hay cambios estructurales)

### 5. Merge a develop

Una vez aprobado, el desarrollador hace merge:

```bash
# Se mergea con --no-ff para mantener historial
git checkout develop
git merge --no-ff feature/JIRA-1234-nueva-funcionalidad
git push origin develop
```

## FASE 3: Despliegue Automático a DEV

### ¿Qué pasa cuando se hace merge a develop?

**Automáticamente se dispara un pipeline que:**

#### Paso 1: Build de la aplicación
- Compila el proyecto en modo Release
- Genera el paquete de despliegue (.zip)

#### Paso 2: Deploy a servidor DEV
- Detiene el sitio IIS en DEV
- Copia los archivos nuevos al servidor
- Actualiza el Web.config con configuración de DEV
- Inicia el sitio IIS nuevamente

#### Paso 3: Ejecuta migración de base de datos
```bash
# Si hay cambios en BD, ejecuta migrations
Update-Database -ConnectionString "DEV_DB_ConnectionString"
```

#### Paso 4: Health Check
```bash
# Verifica que el sitio está funcionando
curl https://dev.myapp.com/health
# Debe retornar 200 OK
```

#### Paso 5: Pruebas de integración
- Ejecuta pruebas de integración contra DEV
- Pruebas que usan base de datos real, APIs externas, etc.

**Tiempo estimado: 10-15 minutos**

### ¿Qué validaciones se hacen en DEV?

✅ El sitio responde correctamente
✅ Las funcionalidades principales funcionan
✅ Las pruebas de integración pasan
✅ No hay errores en los logs

**El equipo de desarrollo verifica manualmente durante 1-2 días que todo funciona bien en DEV**

## FASE 4: Pull Request a Main (para ir a Staging)

### 1. Cuando DEV está estable

Después de 1-2 días probando en DEV sin problemas:

```bash
# Se crea una rama de release
git checkout -b release/v1.5.0 develop
git push origin release/v1.5.0
```

### 2. Crear PR de release → main

**Este PR es más crítico, requiere:**
- ✅ Technical Lead aprueba
- ✅ QA Manager aprueba
- ✅ Product Owner aprueba

### 3. Pipeline de validación más exhaustivo

**Se ejecutan MÁS pruebas:**
- Todas las pruebas unitarias
- Todas las pruebas de integración
- Pruebas de regresión (casos críticos del sistema)
- Análisis de seguridad completo (OWASP, vulnerabilidades)

**Tiempo estimado: 20-30 minutos**

### 4. Merge a main

```bash
git checkout main
git merge --no-ff release/v1.5.0
git tag v1.5.0
git push origin main --tags
```

## FASE 5: Despliegue a Staging

### ¿Qué pasa cuando se mergea a main?

**Automáticamente se dispara el pipeline de Staging:**

#### Paso 1: Build de producción
- Compila en modo Release con optimizaciones
- Genera símbolos de depuración (para diagnóstico)
- Empaqueta la aplicación

#### Paso 2: Backup de Staging actual
```bash
# Antes de desplegar, hace backup por si algo sale mal
Copy-Item "\\staging-server\MyApp" "\\backup-server\MyApp_20250124" -Recurse
```

#### Paso 3: Deploy a Staging
- Detiene el sitio IIS en Staging
- Despliega nueva versión
- Actualiza Web.config con configuración de Staging
- Ejecuta scripts de migración de base de datos
- Inicia el sitio IIS

#### Paso 4: Smoke Tests
Ejecuta pruebas rápidas para verificar que lo básico funciona:
```csharp
// Ejemplos de smoke tests
- Login funciona
- Página principal carga
- API responde
- Base de datos conecta
```

#### Paso 5: Pruebas automatizadas completas

**Pruebas de UI con Selenium:**
```csharp
[TestMethod]
public void DeberiaPermitirLoginConUsuarioValido()
{
    _driver.Navigate().GoToUrl("https://staging.myapp.com");
    _driver.FindElement(By.Id("username")).SendKeys("test@example.com");
    _driver.FindElement(By.Id("password")).SendKeys("password123");
    _driver.FindElement(By.Id("btnLogin")).Click();
    
    Assert.IsTrue(_driver.Url.Contains("/dashboard"));
}
```

**Pruebas de rendimiento:**
- Simula 50 usuarios concurrentes
- Verifica que tiempos de respuesta sean < 2 segundos
- Verifica que no hay memory leaks

**Tiempo total del deployment: 30-45 minutos**

### ¿Qué se valida en Staging?

**Validaciones automáticas (por el pipeline):**
- ✅ Smoke tests pasan
- ✅ Pruebas de UI pasan
- ✅ Performance es aceptable
- ✅ No hay errores en logs

**Validaciones manuales (por el equipo):**

**QA hace pruebas exhaustivas:**
- Todos los flujos de usuario principales
- Casos edge y negativos
- Integración con sistemas externos
- Compatibilidad cross-browser

**Product Owner valida:**
- Las funcionalidades nuevas cumplen requisitos
- La experiencia de usuario es correcta
- Los reportes/dashboards muestran info correcta

**Equipo técnico verifica:**
- Logs no muestran errores
- Performance es aceptable
- Métricas de monitoreo son normales

**Duración típica en Staging: 2-5 días de pruebas**

## FASE 6: Aprobación para Producción

### Gate de aprobación manual

**Cuando Staging está estable, se solicita aprobación para producción:**

**Aprobadores obligatorios:**
1. **Technical Lead** - Valida que técnicamente todo está correcto
2. **Product Owner** - Valida que cumple requisitos de negocio
3. **QA Manager** - Valida que todas las pruebas pasaron

**Aprobadores opcionales (según el cambio):**
- CTO/Director de Tecnología (si hay cambios arquitectónicos grandes)
- Seguridad (si hay cambios de seguridad/autenticación)

### Checklist de aprobación

```markdown
## Checklist Pre-Producción

### Técnico
- [ ] Todas las pruebas pasaron en Staging
- [ ] Performance validada (< 2 seg tiempos respuesta)
- [ ] No hay errores críticos en logs de Staging
- [ ] Base de datos tiene backup reciente
- [ ] Plan de rollback definido y probado

### Funcional
- [ ] Todas las funcionalidades validadas por QA
- [ ] Product Owner aprueba cambios
- [ ] Documentación actualizada
- [ ] Release notes preparadas

### Operacional
- [ ] Ventana de mantenimiento agendada
- [ ] Equipo de soporte notificado
- [ ] Monitoreo preparado para despliegue
- [ ] Contactos de emergencia disponibles
```

**Solo cuando TODOS aprueban → se puede desplegar a producción**

## FASE 7: Despliegue a Producción

### Estrategia: Blue-Green Deployment

**Concepto:** Tener dos entornos idénticos (Blue y Green). El activo es Blue, se despliega en Green y se hace el switch.

```
Antes del deploy:
Blue (ACTIVO) ← usuarios aquí
Green (INACTIVO)

Durante el deploy:
Blue (ACTIVO) ← usuarios siguen aquí
Green (DESPLEGANDO) ← nueva versión

Después de validar:
Blue (INACTIVO) ← versión anterior
Green (ACTIVO) ← usuarios se cambian aquí (nueva versión)
```

### Pasos del despliegue a producción

#### ANTES DEL DEPLOY (Pre-deployment)

**1. Verificar ventana de mantenimiento**
```bash
# Solo se despliega fuera de horario laboral
# Ejemplo: Viernes 22:00 - Sábado 02:00
```

**2. Crear backups completos**

```bash
# Backup de base de datos
BACKUP DATABASE [MyAppDB] 
TO DISK = 'D:\Backups\MyApp_PROD_20250124_2200.bak'
WITH COMPRESSION

# Backup de aplicación
Compress-Archive -Path "C:\inetpub\MyApp" 
               -DestinationPath "\\backup\MyApp_PROD_20250124_2200.zip"
```

**3. Notificar a usuarios (si aplica)**
```
Sistema en mantenimiento
Horario: 22:00 - 24:00
Disculpe las molestias
```

#### DURANTE EL DEPLOY

**1. Desplegar en Green (slot inactivo)**

```bash
# Extraer nueva versión en Green
Expand-Archive -Path "MyApp_v1.5.0.zip" -DestinationPath "C:\inetpub\MyApp-Green"

# Configurar Web.config de producción
Copy-Item "\\config\production.Web.config" "C:\inetpub\MyApp-Green\Web.config"
```

**2. Actualizar base de datos**

```bash
# Ejecutar migrations
Update-Database -ConnectionString "PROD_ConnectionString" -TargetMigration "Latest"
```

**3. Probar en Green (sin usuarios)**

```bash
# Configurar IIS temporal para probar Green
# Puerto temporal 8080 solo accesible internamente

# Health check
curl http://localhost:8080/health
# Debe retornar 200 OK

# Smoke tests críticos
vstest.console.exe SmokeTests.dll --filter "Priority=Critical"
```

**4. Hacer el switch (Go Live)**

```bash
# En IIS, cambiar el path físico del sitio:
# De: C:\inetpub\MyApp-Blue (versión vieja)
# A: C:\inetpub\MyApp-Green (versión nueva)

# Reciclar Application Pool
Restart-WebAppPool -Name "MyApp"

# Los usuarios ahora ven la nueva versión
```

**Downtime total: 2-5 minutos** (solo durante el switch)

#### DESPUÉS DEL DEPLOY (Post-deployment)

**1. Monitoreo intensivo (primeros 30 minutos)**

```bash
# Cada minuto verificar:
# - Health check responde OK
# - No hay errores en logs
# - Tiempos de respuesta normales
# - No hay picos de CPU/memoria
```

**2. Pruebas de humo en producción**

```csharp
// Smoke tests mínimos en PROD
- Login funciona
- Funcionalidades críticas responden
- No hay errores JavaScript en consola
```

**3. Validar métricas de negocio**

```bash
# Verificar que las transacciones funcionan:
# - Se pueden crear pedidos
# - Se procesan pagos
# - Se envían emails
```

**4. Si todo está bien → Mantener nueva versión**

```bash
# Después de 2-4 horas sin problemas:
# - Eliminar backup de Blue
# - Confirmar deployment exitoso
# - Notificar al equipo
```

### ¿Qué pasa si algo falla? → ROLLBACK

**Criterios para hacer rollback automático:**

❌ Health check falla 3 veces consecutivas
❌ Errores críticos en logs (> 10 en 5 minutos)
❌ Tiempo de respuesta > 10 segundos
❌ Tasa de error > 5%

**Proceso de rollback (5-10 minutos):**

```bash
# 1. Cambiar IIS de nuevo a Blue
Set-ItemProperty "IIS:\Sites\MyApp" -Name physicalPath -Value "C:\inetpub\MyApp-Blue"

# 2. Reciclar Application Pool
Restart-WebAppPool -Name "MyApp"

# 3. Restaurar BD si es necesario
RESTORE DATABASE [MyAppDB] 
FROM DISK = 'D:\Backups\MyApp_PROD_20250124_2200.bak'
WITH REPLACE

# 4. Verificar que funciona
curl https://www.myapp.com/health
```

**Después del rollback:**
- Investigar qué falló
- Corregir en develop
- Volver a pasar por todo el proceso

## Resumen visual del flujo

```
┌─────────────────────────────────────────────────────────────┐
│ DESARROLLADOR                                               │
│ 1. Crea rama feature/xxx                                    │
│ 2. Desarrolla + pruebas unitarias                          │
│ 3. Commit + Push                                            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ PULL REQUEST → develop                                      │
│ 1. Build automático                                         │
│ 2. Pruebas unitarias (bloqueante)                          │
│ 3. SonarQube (bloqueante si hay bugs)                      │
│ 4. Cobertura (bloqueante si < 70%)                         │
│ 5. Code Review (2 aprobaciones)                            │
│ 6. Merge                                                    │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ ENTORNO DEV (automático al mergear a develop)              │
│ 1. Build                                                    │
│ 2. Deploy a servidor DEV                                    │
│ 3. Migración de BD                                          │
│ 4. Pruebas de integración                                  │
│ ⏱️  Duración: 10-15 min                                      │
│ 📅 Validación manual: 1-2 días                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ PULL REQUEST → main (release)                               │
│ 1. Todas las pruebas + regresión                           │
│ 2. Análisis de seguridad                                   │
│ 3. Aprobación Tech Lead + QA + PO                          │
│ 4. Merge → Crea tag v1.5.0                                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ ENTORNO STAGING (automático al mergear a main)             │
│ 1. Build producción                                         │
│ 2. Backup Staging                                           │
│ 3. Deploy a Staging                                         │
│ 4. Smoke tests                                              │
│ 5. Pruebas UI (Selenium)                                    │
│ 6. Pruebas de rendimiento                                  │
│ ⏱️  Duración: 30-45 min                                      │
│ 📅 Validación manual por QA: 2-5 días                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ GATE DE APROBACIÓN                                          │
│ ✅ Technical Lead aprueba                                   │
│ ✅ Product Owner aprueba                                    │
│ ✅ QA Manager aprueba                                       │
│ ✅ Checklist pre-producción completo                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ ENTORNO PRODUCCIÓN (manual, fuera de horario laboral)      │
│ PRE-DEPLOY:                                                 │
│   1. Backup BD + Aplicación                                │
│   2. Notificar usuarios                                    │
│ DEPLOY:                                                     │
│   3. Desplegar en Green (inactivo)                         │
│   4. Actualizar BD                                         │
│   5. Probar Green internamente                             │
│   6. Switch Blue→Green (Go Live) ⚡                        │
│ POST-DEPLOY:                                                │
│   7. Monitoreo intensivo 30 min                            │
│   8. Smoke tests en producción                             │
│   9. Validar métricas                                      │
│ ⏱️  Downtime: 2-5 min (solo durante switch)                 │
│ 🔄 Rollback automático si falla                             │
└─────────────────────────────────────────────────────────────┘
```

## Métricas y criterios de bloqueo

### Tabla resumen de umbrales

| Fase | Métrica | Umbral | Bloquea | Acción si falla |
|------|---------|---------|---------|-----------------|
| **PR → develop** | Compilación | 0 errores | ✅ Sí | Corregir código |
| | Pruebas unitarias | 100% pasan | ✅ Sí | Arreglar tests |
| | Cobertura global | ≥70% | ✅ Sí | Añadir pruebas |
| | Cobertura código nuevo | ≥80% | ✅ Sí | Añadir pruebas |
| | Bugs nuevos (SonarQube) | 0 | ✅ Sí | Corregir bugs |
| | Vulnerabilidades | 0 críticas/altas | ✅ Sí | Corregir vulnerabilidades |
| | Complejidad ciclomática | <15 por método | ✅ Sí | Refactorizar |
| | Code Review | 2 aprobaciones | ✅ Sí | Incorporar feedback |
| **Deploy DEV** | Health check | 200 OK | ✅ Sí | Investigar error |
| | Pruebas integración | 100% pasan | ⚠️ Advertencia | Revisar fallos |
| **PR → main** | Pruebas regresión | 100% pasan | ✅ Sí | Arreglar regresiones |
| | Análisis seguridad | Sin vulnerabilidades | ✅ Sí | Parchar seguridad |
| | Aprobaciones | 3 (Tech Lead, QA, PO) | ✅ Sí | Obtener aprobaciones |
| **Deploy Staging** | Smoke tests | 100% pasan | ✅ Sí | Investigar fallos |
| | Pruebas UI | 100% pasan | ⚠️ Advertencia | Revisar UI |
| | Performance | <2 seg respuesta | ⚠️ Advertencia | Optimizar |
| **Aprobación Producción** | Validación QA | Completa | ✅ Sí | Completar testing |
| | Checklist | 100% completo | ✅ Sí | Completar checklist |
| **Deploy Producción** | Health check | 200 OK (3 veces) | ✅ Sí | ROLLBACK |
| | Errores en logs | <10 en 5 min | ✅ Sí | ROLLBACK |
| | Tiempo respuesta | <10 seg | ✅ Sí | ROLLBACK |
| | Tasa de error | <5% | ✅ Sí | ROLLBACK |

**Leyenda:**
- ✅ Sí = Bloquea automáticamente el paso al siguiente stage
- ⚠️ Advertencia = Requiere revisión manual pero no bloquea

## Tiempos estimados del flujo completo

| Fase | Duración |
|------|----------|
| Desarrollo local | 1-5 días (según complejidad) |
| PR validation pipeline | 10-15 minutos |
| Code review | 2-24 horas |
| Validación en DEV | 1-2 días |
| PR a main validation | 20-30 minutos |
| Validación en Staging | 2-5 días |
| Aprobaciones producción | 1-2 días |
| Deploy a producción | 2-4 horas |
| **TOTAL** | **1-3 semanas** |

## Responsabilidades por rol

### Desarrollador
- Crear código de calidad con pruebas
- Responder a comentarios de code review
- Validar en DEV que funciona correctamente

### Tech Lead
- Revisar arquitectura y decisiones técnicas
- Aprobar PRs complejos
- Aprobar despliegues a producción

### QA
- Ejecutar pruebas manuales en Staging
- Validar que los bugs reportados están corregidos
- Aprobar despliegue a producción

### Product Owner
- Validar que las funcionalidades cumplen requisitos
- Priorizar qué se despliega
- Aprobar despliegue a producción

### DevOps/Operaciones
- Mantener pipelines funcionando
- Monitorear despliegues
- Ejecutar rollbacks si es necesario

## Casos especiales

### Hotfix urgente en producción

Si hay un bug crítico en producción:

```bash
# 1. Crear rama desde main (no desde develop)
git checkout main
git checkout -b hotfix/CRITICAL-bug-login

# 2. Corregir el bug

# 3. PR directo a main (aprobación rápida)
# Solo Tech Lead + 1 revisor

# 4. Deploy directo a producción
# Sin pasar por Staging (excepcional)

# 5. Mergear hotfix de vuelta a develop
git checkout develop
git merge hotfix/CRITICAL-bug-login
```

### Deploy por partes (Feature Flags)

Para funcionalidades grandes:

```csharp
public class ConfiguracionService
{
    public bool NuevaFuncionalidadHabilitada()
    {
        // Se despliega a producción pero desactivada
        return ConfigurationManager.AppSettings["NuevaFuncionalidad"] == "true";
    }
}

// En el código
if (_config.NuevaFuncionalidadHabilitada())
{
    // Nueva funcionalidad
}
else
{
    // Funcionalidad antigua
}
```

Se activa cambiando solo configuración, sin redesplegar.

## Conclusión

Este flujo garantiza:
- ✅ **Calidad**: Múltiples validaciones en cada etapa
- ✅ **Estabilidad**: Pruebas exhaustivas antes de producción
- ✅ **Trazabilidad**: Cada cambio está documentado y aprobado
- ✅ **Seguridad**: Análisis de vulnerabilidades en cada paso
- ✅ **Rollback rápido**: Si algo falla, se puede revertir en minutos

**La clave es ser disciplinado con el proceso. No saltarse pasos aunque haya presión por entregar rápido.**
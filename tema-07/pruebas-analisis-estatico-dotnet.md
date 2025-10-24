# Pruebas de Análisis Estático en .NET 4.8

## ¿Qué es el Análisis Estático?

El análisis estático examina el código fuente sin ejecutarlo, identificando problemas potenciales, violaciones de estándares y vulnerabilidades de seguridad antes de la compilación o ejecución.

### Beneficios principales:
- Detección temprana de errores
- Mejora de la calidad del código
- Cumplimiento de estándares de codificación
- Identificación de vulnerabilidades de seguridad
- Reducción de deuda técnica

---

## Roslyn Analyzers

Los analizadores de Roslyn utilizan la plataforma de compilación de .NET para inspeccionar el código en tiempo real.

### Instalación y configuración

```xml
<!-- En el archivo .csproj -->
<ItemGroup>
  <PackageReference Include="Microsoft.CodeAnalysis.FxCopAnalyzers" Version="3.3.2" />
  <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" Version="5.0.3" />
</ItemGroup>
```

### Ejemplo de uso básico

```csharp
// ❌ Código con problemas detectables
public class CustomerService
{
    public void ProcessCustomer(string name)
    {
        // CA1062: Validar argumentos de métodos públicos
        var upperName = name.ToUpper();
    }
}

// ✅ Código corregido
public class CustomerService
{
    public void ProcessCustomer(string name)
    {
        if (name == null)
            throw new ArgumentNullException(nameof(name));
            
        var upperName = name.ToUpper();
    }
}
```

### Configuración de reglas (.editorconfig)

```ini
# .editorconfig en la raíz del proyecto
root = true

[*.cs]
# CA1062: Validar argumentos públicos
dotnet_diagnostic.CA1062.severity = warning

# CA1031: No capturar excepciones generales
dotnet_diagnostic.CA1031.severity = error

# IDE0005: Eliminar usings innecesarios
dotnet_diagnostic.IDE0005.severity = warning
```

---

## StyleCop Analyzers

StyleCop verifica el cumplimiento de las convenciones de estilo de código de C#.

### Instalación

```xml
<ItemGroup>
  <PackageReference Include="StyleCop.Analyzers" Version="1.1.118" />
</ItemGroup>
```

### Ejemplo de reglas comunes

```csharp
// ❌ SA1200: Las directivas using deben estar dentro del namespace
using System;
using System.Collections.Generic;

namespace MiAplicacion
{
    public class Producto { }
}

// ✅ Correcto
namespace MiAplicacion
{
    using System;
    using System.Collections.Generic;

    public class Producto { }
}
```

### Archivo de configuración (stylecop.json)

```json
{
  "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
  "settings": {
    "documentationRules": {
      "companyName": "Mi Empresa",
      "copyrightText": "Copyright (c) {companyName}. Todos los derechos reservados."
    },
    "orderingRules": {
      "usingDirectivesPlacement": "insideNamespace"
    }
  }
}
```

---

## FxCop (Code Analysis)

FxCop analiza ensamblados compilados en busca de problemas de diseño, localización, rendimiento y seguridad.

### Activación en Visual Studio

```xml
<!-- En el archivo .csproj -->
<PropertyGroup>
  <CodeAnalysisRuleSet>MinimumRecommendedRules.ruleset</CodeAnalysisRuleSet>
  <RunCodeAnalysis>true</RunCodeAnalysis>
</PropertyGroup>
```

### Ejemplo de detección

```csharp
// ❌ CA1822: Los miembros que no acceden a datos de instancia deben ser estáticos
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

// ✅ Correcto
public class Calculator
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

---

## .NET Security Code Analysis

Herramienta específica para detectar vulnerabilidades de seguridad en el código.

### Instalación

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.CodeAnalysis.FxCopAnalyzers" Version="3.3.2" />
</ItemGroup>
```

### Ejemplos de vulnerabilidades detectadas

```csharp
// ❌ CA3075: Procesamiento XML inseguro
public void LoadXml(string xmlData)
{
    XmlDocument doc = new XmlDocument();
    doc.LoadXml(xmlData); // Vulnerable a XXE
}

// ✅ Correcto
public void LoadXml(string xmlData)
{
    XmlDocument doc = new XmlDocument();
    doc.XmlResolver = null; // Previene ataques XXE
    doc.LoadXml(xmlData);
}
```

```csharp
// ❌ CA5350: No usar algoritmos criptográficos débiles
public string HashPassword(string password)
{
    using (var sha1 = SHA1.Create())
    {
        byte[] hash = sha1.ComputeHash(Encoding.UTF8.GetBytes(password));
        return Convert.ToBase64String(hash);
    }
}

// ✅ Correcto
public string HashPassword(string password)
{
    using (var sha256 = SHA256.Create())
    {
        byte[] hash = sha256.ComputeHash(Encoding.UTF8.GetBytes(password));
        return Convert.ToBase64String(hash);
    }
}
```

---

## Integración con MSBuild

### Configuración para ejecución automática

```xml
<PropertyGroup>
  <!-- Tratar warnings como errores -->
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  
  <!-- Nivel de warning -->
  <WarningLevel>4</WarningLevel>
  
  <!-- Code Analysis -->
  <RunCodeAnalysis>true</RunCodeAnalysis>
  <CodeAnalysisRuleSet>AllRules.ruleset</CodeAnalysisRuleSet>
</PropertyGroup>
```

### Ejecución desde línea de comandos

```bash
# Compilar con análisis de código
msbuild MiProyecto.sln /p:RunCodeAnalysis=true

# Ver solo warnings y errores
msbuild MiProyecto.sln /v:quiet /clp:ErrorsOnly
```

---

## Supresión de Reglas

### Usando atributos

```csharp
using System.Diagnostics.CodeAnalysis;

[SuppressMessage("Microsoft.Performance", "CA1822:MarkMembersAsStatic", 
    Justification = "Necesario para compatibilidad con interfaces")]
public class LegacyService
{
    public void ProcessData()
    {
        // Lógica que no puede ser estática
    }
}
```

### Usando directivas de preprocesador

```csharp
#pragma warning disable CA1062 // Validar argumentos
public void ProcessItem(string item)
{
    // Código legacy donde la validación se hace en otro lugar
    Console.WriteLine(item.ToUpper());
}
#pragma warning restore CA1062
```

---

## Mejores Prácticas

### 1. Configuración gradual
- Comenzar con reglas de severidad baja
- Incrementar gradualmente la rigurosidad
- No activar todas las reglas simultáneamente

### 2. Ruleset personalizado

```xml
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="Reglas Personalizadas" ToolsVersion="16.0">
  <Rules AnalyzerId="Microsoft.CodeAnalysis.CSharp" RuleNamespace="Microsoft.CodeAnalysis.CSharp">
    <Rule Id="CA1062" Action="Error" />
    <Rule Id="CA1031" Action="Warning" />
    <Rule Id="CA1822" Action="Info" />
  </Rules>
</RuleSet>
```

### 3. Integración en CI/CD

```yaml
# Ejemplo para Azure DevOps
steps:
- task: DotNetCoreCLI@2
  displayName: 'Build con análisis estático'
  inputs:
    command: 'build'
    arguments: '/p:RunCodeAnalysis=true /p:TreatWarningsAsErrors=true'
```

### 4. Revisión regular
- Revisar warnings periódicamente
- No suprimir reglas sin justificación
- Documentar excepciones legítimas
- Mantener configuraciones actualizadas

---

## Herramientas Complementarias

### SonarQube
Plataforma completa de análisis estático con dashboard web.

```bash
# Análisis con SonarScanner
SonarScanner.MSBuild.exe begin /k:"proyecto-key" /d:sonar.host.url="http://localhost:9000"
msbuild /t:Rebuild
SonarScanner.MSBuild.exe end
```

### NDepend
Herramienta avanzada para análisis de arquitectura y dependencias.

### ReSharper Command Line Tools
Análisis desde consola con las capacidades de ReSharper.

```bash
inspectcode.exe MiSolucion.sln -o=resultado.xml
```

---

## Métricas Clave

### Complejidad Ciclomática
- Mide la complejidad lógica del código
- Valor recomendado: ≤ 10 por método

### Índice de Mantenibilidad
- Rango: 0-100
- Verde (≥ 20): Buena mantenibilidad
- Amarillo (10-19): Moderada
- Rojo (< 10): Baja mantenibilidad

### Acoplamiento de Clases
- Mide dependencias entre clases
- Menor acoplamiento = mayor testabilidad

---

## Ejemplo Completo de Proyecto

```csharp
namespace MiAplicacion.Services
{
    using System;
    using System.Diagnostics.CodeAnalysis;

    /// <summary>
    /// Servicio para cálculos de descuentos
    /// </summary>
    public class DiscountService
    {
        /// <summary>
        /// Calcula el descuento aplicable a un monto
        /// </summary>
        /// <param name="amount">Monto base</param>
        /// <param name="percentage">Porcentaje de descuento</param>
        /// <returns>Monto con descuento aplicado</returns>
        /// <exception cref="ArgumentOutOfRangeException">
        /// Se lanza si el porcentaje es inválido
        /// </exception>
        public decimal CalculateDiscount(decimal amount, decimal percentage)
        {
            if (percentage < 0 || percentage > 100)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(percentage), 
                    "El porcentaje debe estar entre 0 y 100");
            }

            return amount * (1 - percentage / 100);
        }
    }
}
```

---

## Resumen

El análisis estático es fundamental para:
- **Prevenir errores** antes de la ejecución
- **Mantener calidad** consistente del código
- **Detectar vulnerabilidades** de seguridad
- **Facilitar mantenimiento** a largo plazo
- **Automatizar revisiones** de código

La clave está en integrar estas herramientas en el flujo de trabajo diario y en los pipelines de CI/CD, configurándolas de manera progresiva según las necesidades del equipo y del proyecto.

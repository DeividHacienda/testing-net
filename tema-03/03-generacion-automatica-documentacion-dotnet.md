# Generación Automática de Documentación en .NET: DocFX vs Sandcastle

## Introducción

La documentación técnica es un componente esencial en cualquier proyecto de software profesional. En el ecosistema .NET, los **XML Documentation Comments** proporcionan metadatos valiosos sobre el código, pero necesitamos herramientas que transformen estos comentarios en documentación navegable y presentable. Las dos soluciones principales son **DocFX** y **Sandcastle Help File Builder (SHFB)**.

Este documento presenta una comparativa exhaustiva de ambas herramientas para ayudarte a elegir la más adecuada para tu proyecto.

---

## DocFX

### ¿Qué es DocFX?

DocFX es una herramienta moderna de código abierto desarrollada por Microsoft para generar documentación de APIs y sitios de documentación estática. Combina documentación de referencia de APIs (generada desde código fuente) con artículos conceptuales escritos en Markdown.

### Características principales

**Ventajas:**
- **Moderna y mantenida activamente**: Desarrollo continuo por parte de Microsoft
- **Múltiples formatos de salida**: HTML estático, PDF (mediante plugins)
- **Integración con Markdown**: Permite mezclar documentación de API con artículos conceptuales
- **Soporte multiplataforma**: Funciona en Windows, Linux y macOS
- **Temas personalizables**: Sistema de plantillas flexible basado en Liquid y Mustache
- **Integración nativa con GitHub/Azure DevOps**: Ideal para documentación alojada en la nube
- **Búsqueda integrada**: Funcionalidad de búsqueda client-side sin necesidad de servidor
- **Soporte para múltiples lenguajes**: C#, VB.NET, F#, TypeScript, JavaScript, Python, Java
- **Versionado de documentación**: Facilita la gestión de múltiples versiones
- **API incremental**: Solo regenera lo que ha cambiado, mejorando el rendimiento

**Desventajas:**
- **Curva de aprendizaje**: Requiere familiarización con su estructura de proyecto
- **Configuración inicial más compleja**: Necesita un archivo `docfx.json` detallado
- **Menos opciones de salida nativas**: No genera CHM (Compiled HTML Help) nativamente
- **Documentación fragmentada**: La documentación oficial puede ser inconsistente en algunos aspectos

### Instalación

Para proyectos .NET Framework 4.8, DocFX se instala mediante:

**Descarga directa (recomendado para .NET Framework):**
1. Descargar desde: https://github.com/dotnet/docfx/releases
2. Descomprimir en una carpeta (ej: `C:\Tools\DocFX`)
3. Agregar la ruta al PATH del sistema o ejecutar directamente

**Como paquete NuGet en el proyecto:**
```powershell
Install-Package docfx.console
```

**Mediante Chocolatey:**
```powershell
choco install docfx
```

### Configuración básica

**1. Inicializar un proyecto DocFX:**
```powershell
docfx init -q
```

Esto crea la estructura básica:
```
proyecto/
├── docfx.json          # Archivo de configuración principal
├── index.md            # Página de inicio
├── toc.yml             # Tabla de contenidos
├── api/                # Documentación de API generada
├── articles/           # Artículos conceptuales
└── _site/              # Salida HTML generada
```

**2. Archivo de configuración `docfx.json`:**
```json
{
  "metadata": [
    {
      "src": [
        {
          "files": ["**/*.csproj"],
          "src": "../src"
        }
      ],
      "dest": "api",
      "includePrivateMembers": false,
      "disableGitFeatures": false,
      "disableDefaultFilter": false,
      "properties": {
        "TargetFramework": "net48"
      }
    }
  ],
  "build": {
    "content": [
      {
        "files": ["api/**.yml", "api/index.md"]
      },
      {
        "files": ["articles/**.md", "articles/**/toc.yml", "toc.yml", "*.md"]
      }
    ],
    "resource": [
      {
        "files": ["images/**"]
      }
    ],
    "output": "_site",
    "template": ["default", "modern"],
    "globalMetadata": {
      "_appTitle": "Mi Proyecto .NET Framework 4.8",
      "_appFooter": "© 2025 Mi Empresa",
      "_enableSearch": true
    }
  }
}
```

**3. Generar documentación:**
```powershell
# Generar metadatos desde el código
docfx metadata

# Construir el sitio de documentación
docfx build

# Generar y servir en un servidor local
docfx --serve
```

**Si DocFX no está en el PATH:**
```powershell
# Navegar a la carpeta de DocFX
cd C:\Tools\DocFX

# Ejecutar directamente
.\docfx.exe metadata
.\docfx.exe build
.\docfx.exe --serve
```

### Ejemplo de uso con XML Comments

**Código fuente con documentación:**
```csharp
namespace MiProyecto.Services
{
    /// <summary>
    /// Servicio para gestionar operaciones de usuarios.
    /// </summary>
    /// <remarks>
    /// Este servicio implementa operaciones CRUD completas para usuarios
    /// y se integra con el sistema de autenticación.
    /// </remarks>
    public class UserService : IUserService
    {
        /// <summary>
        /// Obtiene un usuario por su identificador único.
        /// </summary>
        /// <param name="userId">El ID único del usuario.</param>
        /// <returns>
        /// Un objeto <see cref="User"/> si se encuentra, 
        /// o <c>null</c> si no existe.
        /// </returns>
        /// <exception cref="ArgumentException">
        /// Se lanza cuando <paramref name="userId"/> es inválido.
        /// </exception>
        /// <example>
        /// <code>
        /// var service = new UserService();
        /// var user = await service.GetUserByIdAsync(123);
        /// </code>
        /// </example>
        public async Task<User?> GetUserByIdAsync(int userId)
        {
            if (userId <= 0)
                throw new ArgumentException("El ID debe ser positivo", nameof(userId));
            
            return await _repository.FindByIdAsync(userId);
        }
    }
}
```

### Personalización de temas

DocFX permite personalizar completamente la apariencia:

**Crear un tema personalizado:**
```powershell
docfx template export default
```

Esto extrae el tema por defecto que puedes modificar. Los archivos principales son:
- `styles/main.css`: Estilos CSS
- `partials/`: Componentes HTML reutilizables
- `layout/`: Plantillas de página

**Ejemplo de personalización en `docfx.json`:**
```json
{
  "build": {
    "template": ["default", "templates/custom"],
    "globalMetadata": {
      "_appLogoPath": "images/logo.png",
      "_appFaviconPath": "images/favicon.ico",
      "_enableNewTab": true
    }
  }
}
```

---

## Sandcastle Help File Builder (SHFB)

### ¿Qué es Sandcastle?

Sandcastle es una suite de herramientas desarrollada originalmente por Microsoft (ahora mantenida por la comunidad) para generar documentación de APIs en formato de ayuda tradicional. **Sandcastle Help File Builder (SHFB)** es la interfaz gráfica y de construcción más popular para trabajar con Sandcastle.

### Características principales

**Ventajas:**
- **Formatos de salida tradicionales**: CHM (Compiled HTML Help), HTML Help 1.x, MS Help Viewer (.mshc)
- **Interfaz gráfica intuitiva**: GUI completa para configuración (también CLI disponible)
- **Madurez y estabilidad**: Años de desarrollo y refinamiento
- **Documentación exhaustiva**: Amplia documentación y ejemplos
- **Integración con Visual Studio**: Plugin disponible para edición de proyectos
- **Soporte para herencia de documentación**: Copia automática de comentarios desde interfaces y clases base
- **Plugins extensibles**: Sistema de plugins para funcionalidades adicionales
- **Generación de sitios web estáticos**: Además de formatos de ayuda tradicionales

**Desventajas:**
- **Orientado principalmente a Windows**: Aunque existe soporte experimental para .NET Core
- **Interfaz menos moderna**: La GUI puede parecer anticuada comparada con herramientas modernas
- **Tiempos de compilación más largos**: Especialmente en proyectos grandes
- **Mantenimiento comunitario**: No tiene el respaldo activo de Microsoft
- **Menos flexible con Markdown**: No integra artículos conceptuales tan naturalmente como DocFX
- **Curva de aprendizaje de formatos**: CHM y MS Help Viewer son formatos menos comunes hoy en día

### Instalación

**Descarga e instalación:**
1. Descargar desde: https://github.com/EWSoftware/SHFB/releases
2. Ejecutar el instalador (requiere .NET Framework 4.7.2 o superior)
3. Opcionalmente, instalar el plugin de Visual Studio

**Componentes instalados:**
- Sandcastle Help File Builder (GUI)
- Sandcastle Tools
- Reflection Data Manager
- HTML Help Workshop (para generar CHM)

### Configuración básica

**1. Crear un proyecto SHFB:**

Puedes crear un proyecto mediante la GUI o mediante un archivo `.shfbproj` (formato MSBuild):

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" DefaultTargets="Build" 
         xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <!-- Configuración básica del proyecto -->
    <SHFBROOT Condition=" '$(SHFBROOT)' == '' ">$(MSBuildProgramFiles32)\EWSoftware\Sandcastle Help File Builder\</SHFBROOT>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <SchemaVersion>2.0</SchemaVersion>
    <ProjectGuid>{12345678-1234-1234-1234-123456789012}</ProjectGuid>
    <SHFBSchemaVersion>2017.9.26.0</SHFBSchemaVersion>
    
    <!-- Propiedades de documentación -->
    <OutputPath>.\Help\</OutputPath>
    <HtmlHelpName>MiProyecto</HtmlHelpName>
    <Language>es-ES</Language>
    <DocumentationSources>
      <DocumentationSource sourceFile="..\src\MiProyecto\MiProyecto.csproj" />
    </DocumentationSources>
    
    <!-- Formatos de salida -->
    <HelpFileFormat>HtmlHelp1, Website</HelpFileFormat>
    
    <!-- Metadatos -->
    <HelpTitle>Documentación de Mi Proyecto .NET</HelpTitle>
    <CopyrightText>Copyright %28c%29 2025 Mi Empresa</CopyrightText>
    <FeedbackEMailAddress>soporte@miempresa.com</FeedbackEMailAddress>
    
    <!-- Opciones de construcción -->
    <FrameworkVersion>.NET Framework 4.8</FrameworkVersion>
    <PresentationStyle>VS2013</PresentationStyle>
    <NamespaceSummaries>
      <NamespaceSummaryItem name="MiProyecto.Services" isDocumented="True">
        Contiene servicios de negocio principales
      </NamespaceSummaryItem>
    </NamespaceSummaries>
  </PropertyGroup>
  
  <!-- Referencias y elementos adicionales -->
  <ItemGroup>
    <None Include="Content\Welcome.aml" />
    <None Include="Content\GettingStarted.aml" />
  </ItemGroup>
  
  <Import Project="$(SHFBROOT)\SandcastleHelpFileBuilder.targets" />
</Project>
```

**2. Compilar la documentación:**

**Desde la GUI:**
- Abrir el archivo `.shfbproj`
- Hacer clic en "Build" > "Build Project"

**Desde la línea de comandos:**
```cmd
MSBuild MiProyecto.shfbproj /p:Configuration=Release
```

**Desde un script de CI/CD:**
```cmd
"%ProgramFiles(x86)%\MSBuild\14.0\Bin\MSBuild.exe" ^
  Documentation.shfbproj ^
  /p:Configuration=Release ^
  /p:OutputPath=.\Output\
```

### Ejemplo con XML Comments

Sandcastle procesa los mismos XML Comments que DocFX:

```csharp
namespace MiProyecto.Data
{
    /// <summary>
    /// Repositorio genérico para operaciones de acceso a datos.
    /// </summary>
    /// <typeparam name="T">El tipo de entidad a gestionar.</typeparam>
    /// <remarks>
    /// <para>
    /// Este repositorio implementa el patrón Repository y proporciona
    /// operaciones CRUD estándar para cualquier entidad.
    /// </para>
    /// <para>
    /// <b>Nota:</b> Todas las operaciones son asíncronas para mejorar
    /// el rendimiento en aplicaciones web.
    /// </para>
    /// </remarks>
    public class Repository<T> : IRepository<T> where T : class
    {
        /// <summary>
        /// Añade una nueva entidad a la base de datos.
        /// </summary>
        /// <param name="entity">La entidad a añadir.</param>
        /// <returns>
        /// Una tarea que representa la operación asíncrona.
        /// El resultado contiene la entidad añadida con su ID generado.
        /// </returns>
        /// <exception cref="ArgumentNullException">
        /// Se lanza cuando <paramref name="entity"/> es <c>null</c>.
        /// </exception>
        /// <exception cref="DbUpdateException">
        /// Se lanza cuando ocurre un error al guardar en la base de datos.
        /// </exception>
        /// <seealso cref="UpdateAsync"/>
        /// <seealso cref="DeleteAsync"/>
        public async Task<T> AddAsync(T entity)
        {
            if (entity == null)
                throw new ArgumentNullException(nameof(entity));
            
            await _context.Set<T>().AddAsync(entity);
            await _context.SaveChangesAsync();
            return entity;
        }
    }
}
```

### Temas y personalización

SHFB incluye varios estilos de presentación predefinidos:

- **VS2013**: Estilo similar a la documentación de Visual Studio 2013
- **VS2010**: Estilo clásico de Visual Studio 2010
- **Open XML**: Para documentación en formato Office

**Personalización mediante archivos de configuración:**
```xml
<PropertyGroup>
  <PresentationStyle>VS2013</PresentationStyle>
  <HelpFileFormat>Website</HelpFileFormat>
  
  <!-- Personalización de colores y estilos -->
  <HeaderText>Mi Proyecto .NET</HeaderText>
  <FooterText>Documentación interna</FooterText>
  
  <!-- Logo personalizado -->
  <HelpFileVersion>1.0.0.0</HelpFileVersion>
</PropertyGroup>
```

---

## Comparativa detallada

### Tabla comparativa

| Característica | DocFX | Sandcastle (SHFB) |
|----------------|-------|-------------------|
| **Desarrollador** | Microsoft (activo) | Comunidad (Eric Woodruff) |
| **Licencia** | MIT (Open Source) | MIT (Open Source) |
| **Plataformas** | Windows, Linux, macOS | Principalmente Windows |
| **Frameworks soportados** | .NET Framework, .NET Core, .NET 5+ | .NET Framework principalmente |
| **Formatos de salida** | HTML, JSON, PDF (plugin) | CHM, HTML, MS Help Viewer, Website |
| **Integración Markdown** | ✅ Excelente | ⚠️ Limitada |
| **Interfaz gráfica** | ❌ Solo CLI | ✅ GUI completa |
| **Velocidad de compilación** | ⚡ Rápida (incremental) | 🐌 Más lenta |
| **Curva de aprendización** | Media-Alta | Media |
| **Temas modernos** | ✅ Sí, personalizables | ⚠️ Estilos más tradicionales |
| **Versionado de docs** | ✅ Soporte nativo | ⚠️ Requiere configuración manual |
| **Búsqueda integrada** | ✅ Client-side | ✅ Depende del formato |
| **CI/CD friendly** | ✅ Excelente | ✅ Buena (requiere MSBuild) |
| **Documentación conceptual** | ✅ Markdown nativo | ⚠️ Formato MAML (XML) |
| **Soporte .NET Framework 4.8** | ✅ Compatible | ✅ Nativo |
| **Actualización activa** | ✅ Desarrollo continuo | ⚠️ Actualizaciones menos frecuentes |

### Rendimiento

**DocFX:**
- Compilación incremental: solo regenera archivos modificados
- Proyectos grandes (1000+ tipos): 2-5 minutos
- Ideal para regeneración frecuente en desarrollo

**Sandcastle:**
- Compilación completa en cada ejecución
- Proyectos grandes: 5-15 minutos
- Mejor para compilaciones nocturnas o releases

### Integración en CI/CD

**Ejemplo de pipeline con DocFX (Azure DevOps para .NET Framework 4.8):**
```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

steps:
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    restoreSolution: '**/*.sln'
  displayName: 'Restaurar paquetes NuGet'

- task: VSBuild@1
  inputs:
    solution: '**/*.sln'
    msbuildArgs: '/p:Configuration=Release'
  displayName: 'Compilar solución'

- powershell: |
    # Descargar DocFX si no está disponible
    if (-not (Test-Path ".\docfx\docfx.exe")) {
      Invoke-WebRequest -Uri "https://github.com/dotnet/docfx/releases/download/v2.70.0/docfx.zip" -OutFile "docfx.zip"
      Expand-Archive -Path "docfx.zip" -DestinationPath ".\docfx"
    }
    
    # Generar documentación
    .\docfx\docfx.exe metadata
    .\docfx\docfx.exe build
  displayName: 'Generar documentación con DocFX'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '_site'
    ArtifactName: 'documentation'
```

**Ejemplo de pipeline con Sandcastle (Azure DevOps):**
```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

steps:
- task: NuGetToolInstaller@1

- task: MSBuild@1
  inputs:
    solution: 'Documentation.shfbproj'
    configuration: 'Release'
  displayName: 'Compilar documentación con SHFB'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'Help'
    ArtifactName: 'documentation'
```

### Casos de uso recomendados

**Usa DocFX si:**
- Necesitas documentación moderna para sitios web estáticos
- Quieres combinar documentación de API con artículos conceptuales en Markdown
- Trabajas con proyectos .NET Framework 4.8 y quieres flexibilidad futura
- Requieres integración con GitHub Pages, Azure Static Web Apps, o similares
- Necesitas soporte para múltiples versiones de documentación
- Buscas una solución con desarrollo activo y soporte de Microsoft
- Tu equipo prefiere trabajar principalmente con archivos de texto (Markdown, YAML)
- Tu proyecto puede necesitar migrar a .NET Core/.NET 5+ en el futuro

**Usa Sandcastle si:**
- Necesitas generar archivos CHM (Compiled HTML Help) tradicionales
- Trabajas exclusivamente en Windows con .NET Framework
- Prefieres una interfaz gráfica para configurar la documentación
- Ya tienes experiencia con Sandcastle y tu pipeline está establecido
- Necesitas formatos específicos como MS Help Viewer para integración con Visual Studio
- Tu documentación es principalmente de referencia de API sin muchos artículos conceptuales
- Tu proyecto .NET Framework 4.8 permanecerá en esa versión a largo plazo

---

## Flujo de trabajo completo

### Con DocFX

**1. Preparar el código fuente:**
```csharp
/// <summary>
/// Calcula el precio total con impuestos incluidos.
/// </summary>
/// <param name="basePrice">Precio base sin impuestos.</param>
/// <param name="taxRate">Tasa de impuesto (ej: 0.21 para 21%).</param>
/// <returns>El precio total con impuestos.</returns>
public decimal CalculateTotalPrice(decimal basePrice, decimal taxRate)
{
    return basePrice * (1 + taxRate);
}
```

**2. Configurar el proyecto:**
```powershell
# Inicializar
docfx init -q

# Editar docfx.json para apuntar a tu código fuente
# Asegurarse de especificar "TargetFramework": "net48"
```

**3. Crear artículos conceptuales:**
```markdown
# articles/getting-started.md

# Comenzando con Mi Biblioteca

Esta guía te ayudará a empezar a usar nuestra biblioteca.

## Instalación

Agregar referencia desde NuGet Package Manager:
```
Install-Package MiBiblioteca
```

O desde el archivo .csproj:
```xml
<PackageReference Include="MiBiblioteca" Version="1.0.0" />
```

## Uso básico

```csharp
var servicio = new MiServicio();
var resultado = servicio.Procesar();
```
```

**4. Generar y publicar:**
```powershell
docfx build
# Los archivos están en _site/

# Publicar en GitHub Pages usando Git Bash o comandos Git
git subtree push --prefix _site origin gh-pages

# O publicar en IIS copiando la carpeta _site
Copy-Item -Path "_site\*" -Destination "C:\inetpub\wwwroot\docs" -Recurse
```

### Con Sandcastle

**1. Preparar el código fuente** (igual que con DocFX)

**2. Crear proyecto SHFB:**
- Abrir Sandcastle Help File Builder
- File > New Project
- Agregar fuentes de documentación (archivos .csproj o .dll + .xml)

**3. Configurar opciones:**
- Establecer formato de salida (Website recomendado para uso moderno)
- Configurar namespace summaries
- Añadir contenido conceptual (archivos MAML si es necesario)

**4. Compilar:**
- Build > Build Project
- Los archivos están en la carpeta Help/

**5. Publicar:**
- Copiar contenido de Help/ a tu servidor web o directorio de distribución

---

## Migración entre herramientas

### De Sandcastle a DocFX

**Pasos recomendados:**

1. **Mantener XML Comments intactos**: Ambas herramientas los usan igual
2. **Convertir contenido MAML a Markdown**:
   - MAML es XML, Markdown es mucho más simple
   - Herramientas como Pandoc pueden ayudar parcialmente
3. **Reconfigurar el proyecto**: Crear `docfx.json` basándose en la configuración SHFB
4. **Adaptar el pipeline de CI/CD**: Migrar de MSBuild a comandos DocFX

**Script de ayuda para conversión de estructura:**
```powershell
# Ejemplo: copiar archivos relevantes
$sourceDir = ".\SandcastleProject\"
$targetDir = ".\DocFxProject\"

# Crear estructura DocFX
docfx init -q -o $targetDir

# Copiar proyectos de código
Copy-Item "$sourceDir\*.csproj" "$targetDir\src\" -Recurse
```

### De DocFX a Sandcastle

**Pasos recomendados:**

1. **XML Comments ya están listos**: No requieren cambios
2. **Convertir Markdown a MAML**: Más complejo, considerar si realmente necesitas MAML
3. **Crear proyecto SHFB**: Configurar manualmente basándote en `docfx.json`
4. **Adaptar pipeline**: Cambiar de comandos DocFX a MSBuild

---

## Mejores prácticas comunes

Independientemente de la herramienta elegida:

### 1. Escribir XML Comments de calidad

```csharp
/// <summary>
/// NO: "Obtiene usuario" 
/// SÍ: "Obtiene un usuario de la base de datos por su identificador único."
/// </summary>
/// <param name="id">NO: "El id" | SÍ: "El identificador único del usuario en la base de datos."</param>
/// <returns>
/// SÍ: "Un objeto User si se encuentra, o null si no existe ningún usuario con ese ID."
/// </returns>
/// <exception cref="ArgumentException">
/// SÍ: "Se lanza cuando el ID proporcionado es menor o igual a cero."
/// </exception>
public User GetUserById(int id) { }
```

### 2. Documentar excepciones

Siempre documenta las excepciones que pueden lanzarse:
```csharp
/// <exception cref="ArgumentNullException">Cuando <paramref name="connection"/> es null.</exception>
/// <exception cref="InvalidOperationException">Cuando la conexión no está abierta.</exception>
/// <exception cref="SqlException">Cuando ocurre un error de base de datos.</exception>
```

### 3. Usar referencias cruzadas

```csharp
/// <summary>
/// Similar a <see cref="GetUserById"/>, pero busca por email.
/// Ver también <seealso cref="FindUsersByName"/> para búsquedas por nombre.
/// </summary>
```

### 4. Incluir ejemplos de código

```csharp
/// <example>
/// <code>
/// var repository = new UserRepository();
/// var user = await repository.GetUserByIdAsync(42);
/// if (user != null)
/// {
///     Console.WriteLine($"Usuario encontrado: {user.Name}");
/// }
/// </code>
/// </example>
```

### 5. Automatizar en el pipeline

**Generar documentación en cada commit a main:**
- Ayuda a detectar errores de documentación temprano
- Mantiene la documentación siempre actualizada
- Facilita la revisión de cambios en la documentación

### 6. Revisar la documentación generada

- No confíes ciegamente en la generación automática
- Revisa periódicamente la salida HTML
- Verifica que los enlaces funcionen correctamente
- Asegúrate de que la navegación sea intuitiva

### 7. Versionado de documentación

Mantén documentación para múltiples versiones de tu API:
- Usuarios pueden estar usando versiones antiguas
- Facilita la migración mostrando diferencias entre versiones
- DocFX tiene mejor soporte nativo para esto

---

## Conclusiones y recomendaciones

### Resumen ejecutivo

**Para proyectos .NET Framework 4.8 nuevos**: **DocFX** es la opción recomendada debido a su modernidad, flexibilidad, mejor integración con Markdown, y soporte continuo de Microsoft. Además, facilita una migración futura si el proyecto evoluciona a .NET Core o versiones posteriores.

**Para proyectos existentes con Sandcastle**: Evalúa si realmente necesitas migrar. Si tu documentación actual funciona bien, cumple su propósito, y el proyecto no planea migrar de .NET Framework 4.8, la migración puede no justificar el esfuerzo.

**Para documentación exclusiva CHM**: **Sandcastle** sigue siendo la mejor opción para generar archivos de ayuda tradicionales CHM si este formato es un requisito específico (por ejemplo, para distribución offline o integración con aplicaciones de escritorio).

### Criterios de decisión

| Criterio | Elige DocFX | Elige Sandcastle |
|----------|-------------|------------------|
| Proyecto nuevo .NET Framework 4.8 | ✅ | ⚠️ |
| Hosting web moderno | ✅ | ⚠️ |
| Solo desarrollo en Windows | ✅ | ✅ |
| Formato CHM requerido | ❌ | ✅ |
| Interfaz gráfica necesaria | ❌ | ✅ |
| Documentación conceptual extensa | ✅ | ⚠️ |
| Proyecto legacy sin cambios futuros | ⚠️ | ✅ |
| Migración futura a .NET Core/5+ | ✅ | ❌ |
| Equipo familiarizado con herramienta | Depende | Depende |
| Integración con Visual Studio | ✅ | ✅ |

### Tendencias futuras

La industria se mueve hacia:
- Documentación como código (Docs as Code)
- Formatos web estáticos sobre archivos compilados
- Integración con Git y control de versiones
- Markdown sobre formatos XML propietarios
- Herramientas multiplataforma y cloud-native

DocFX está mejor posicionada en estas tendencias, mientras que Sandcastle representa un enfoque más tradicional pero probado.

---

## Recursos adicionales

### DocFX
- **Sitio oficial**: https://dotnet.github.io/docfx/
- **Repositorio GitHub**: https://github.com/dotnet/docfx
- **Documentación**: https://dotnet.github.io/docfx/docs/
- **Ejemplos**: https://github.com/dotnet/docfx/tree/main/samples

### Sandcastle
- **Sitio oficial SHFB**: https://github.com/EWSoftware/SHFB
- **Documentación**: https://ewsoftware.github.io/SHFB/html/
- **Guías y tutoriales**: https://ewsoftware.github.io/SHFB/html/8c0c97d0-c968-4c15-9fe9-e8f3a443c50a.htm
- **Foro de soporte**: https://github.com/EWSoftware/SHFB/discussions

### Herramientas complementarias
- **XML Documentation**: https://learn.microsoft.com/dotnet/csharp/language-reference/xmldoc/
- **Pandoc** (conversión de documentos): https://pandoc.org/
- **Markdig** (procesador Markdown usado por DocFX): https://github.com/xoofx/markdig

---

## Ejercicios prácticos

### Ejercicio 1: Configurar DocFX en un proyecto existente

1. Toma un proyecto .NET existente con XML Comments
2. Instala DocFX como herramienta global
3. Inicializa un proyecto DocFX
4. Configura `docfx.json` para incluir tu código fuente
5. Genera la documentación y revisa el resultado

### Ejercicio 2: Crear documentación híbrida

1. Crea un proyecto con DocFX
2. Añade al menos 3 artículos conceptuales en Markdown:
   - Guía de inicio rápido
   - Tutorial paso a paso
   - Preguntas frecuentes (FAQ)
3. Configura el `toc.yml` para una navegación clara
4. Personaliza el tema para incluir el logo de tu empresa

### Ejercicio 3: Comparar salidas

1. Toma el mismo código fuente
2. Genera documentación con DocFX
3. Genera documentación con Sandcastle
4. Compara:
   - Apariencia visual
   - Facilidad de navegación
   - Tiempo de generación
   - Estructura de archivos resultantes

### Ejercicio 4: Automatización en CI/CD

1. Configura un pipeline en GitHub Actions o Azure DevOps
2. Automatiza la generación de documentación con DocFX
3. Despliega automáticamente a GitHub Pages o Azure Static Web Apps
4. Configura para que solo se ejecute en commits a `main`

---

## Glosario

- **CHM (Compiled HTML Help)**: Formato de archivo de ayuda de Microsoft, compilado y ejecutable en Windows. Archivo con extensión .chm que contiene documentación HTML comprimida con índice y búsqueda integrados.

- **MAML (Microsoft Assistance Markup Language)**: Lenguaje XML para documentación conceptual usado por Sandcastle. Define la estructura de artículos, tutoriales y guías de forma más verbosa que Markdown.

- **XML Documentation Comments**: Comentarios especiales en código C# que comienzan con `///` y utilizan etiquetas XML como `<summary>`, `<param>`, `<returns>` para documentar código. El compilador puede generar archivos XML con esta información.

- **MSBuild**: Sistema de compilación de Microsoft para proyectos .NET. Tanto DocFX como Sandcastle pueden integrarse con MSBuild para automatizar la generación de documentación.

- **API Reference**: Documentación técnica generada automáticamente desde el código fuente que describe clases, métodos, propiedades y otros miembros de una API.

- **Conceptual Documentation**: Documentación escrita manualmente que explica conceptos, tutoriales, guías de inicio y mejores prácticas (complementa la documentación de API).

- **Static Site Generator**: Herramienta que genera un sitio web completo con archivos HTML, CSS y JavaScript estáticos que pueden hospedarse sin necesidad de un servidor backend.

- **Incremental Build**: Proceso de compilación que solo regenera los archivos que han cambiado desde la última compilación, mejorando significativamente los tiempos de generación.

- **SHFB (Sandcastle Help File Builder)**: Interfaz gráfica y sistema de construcción para Sandcastle que simplifica la creación de proyectos de documentación.

- **MS Help Viewer**: Formato de ayuda de Microsoft usado en Visual Studio y otras herramientas de desarrollo. Archivos con extensión .mshc que se integran con el sistema de ayuda de Visual Studio.

- **Markdown**: Lenguaje de marcado ligero y fácil de leer que se convierte en HTML. Usado ampliamente para documentación técnica por su simplicidad.

- **YAML (YAML Ain't Markup Language)**: Formato de serialización de datos legible por humanos, usado por DocFX para archivos de configuración y tablas de contenidos (TOC).

- **TOC (Table of Contents)**: Tabla de contenidos que define la estructura de navegación de la documentación. En DocFX se define en archivos `toc.yml`.

- **Reflection**: Proceso mediante el cual las herramientas de documentación examinan los ensamblados .NET compilados para extraer información sobre tipos, métodos y sus atributos de documentación XML.

- **CI/CD (Continuous Integration/Continuous Deployment)**: Prácticas de desarrollo que automatizan la compilación, prueba y despliegue de software, incluyendo la generación de documentación.

- **Quality Gate**: Punto de control en un pipeline de CI/CD que verifica que el código cumple ciertos estándares de calidad antes de permitir su despliegue.

- **Namespace Summary**: Descripción general de un espacio de nombres (namespace) que aparece en la documentación generada, explicando su propósito y contenido.

- **IntelliSense**: Característica de autocompletado de Visual Studio que muestra información sobre APIs mientras se escribe código. Se alimenta de los XML Documentation Comments.

- **Roslyn**: Plataforma de compilación de .NET ("compilador como servicio") que incluye APIs para análisis y transformación de código, usada por herramientas de documentación para analizar código fuente.

- **GitHub Pages**: Servicio de hosting gratuito de GitHub para sitios web estáticos, comúnmente usado para alojar documentación generada con DocFX.

- **Azure Static Web Apps**: Servicio de Microsoft Azure para hospedar aplicaciones web estáticas con CI/CD integrado, ideal para documentación generada automáticamente.
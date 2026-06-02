# 🎯 Implementación de Patrones de Diseño y Buenas Prácticas

## Patrones Implementados

### ✅ 1. **Repository Pattern** 
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Data/Repository/IRepository.cs`
- **Función:** Abstrae el acceso a datos
- **Métodos:** GetByIdAsync, GetAllAsync, FindAsync, AddAsync, Update, Delete, Any, GetQueryable
- **Ventaja:** Cambiar de base de datos sin afectar la lógica de negocio

### ✅ 2. **Unit of Work Pattern**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Data/UnitOfWork/IUnitOfWork.cs`
- **Función:** Coordina múltiples repositorios en transacciones atómicas
- **Propiedades:** Users, FormSubmissions (lazy-loaded)
- **Métodos:** SaveChangesAsync, BeginTransactionAsync, CommitTransactionAsync, RollbackTransactionAsync
- **Ventaja:** Consistencia de transacciones y rollback automático

### ✅ 3. **Custom Exceptions**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Helpers/Exceptions/AppException.cs`
- **Tipos:**
  - `AppException` - Excepción base (StatusCode property, Code)
  - `NotFoundException` - 404 (recurso no encontrado)
  - `ValidationException` - 400 (errores de validación con diccionario)
  - `UnauthorizedException` - 401 (sin autenticación/autorización)
  - `ConflictException` - 409 (conflicto de datos)
- **Ventaja:** Código de error consistente y tipado en toda la aplicación

### ✅ 4. **Result Pattern**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Helpers/Results/Result.cs`
- **Clases:** 
  - `Result<T>` - Resultado genérico con datos
  - `Result` - Resultado sin datos
- **Factory Methods:** Success(), Failure()
- **Propiedades:** IsSuccess, Data, Message, Code, Errors
- **Ventaja:** Alternativa funcional a excepciones, mejor rendimiento

### ✅ 5. **Fluent Validation**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Validators/RequestValidators.cs`
- **Validadores:**
  - `LoginRequestValidator` - Username (3-50 chars), Password (6+ chars)
  - `RegisterRequestValidator` - Username, Email, Password (mayús + número), FullName
  - `FormSubmissionRequestValidator` - Nombre, Email, Teléfono (7+ dígitos), Empresa, Asunto, Mensaje (10-2000 chars)
  - `FormFilterRequestValidator` - PageNumber/PageSize, Status validación, Date range
  - `StatusUpdateRequestValidator` - Status debe ser Pendiente/Revisado/Completado
- **Ventaja:** Validaciones declarativas, reutilizables, complejas sin código tedioso

### ✅ 6. **AutoMapper (Object Mapping)**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Mapping/MappingProfile.cs`
- **Mapeos:**
  - `User → LoginResponse` (excluye Token)
  - `User → RegisterResponse`
  - `FormSubmissionRequest → FormSubmission` (establece Status="Pendiente")
  - `FormSubmission → FormSubmissionResponse`
- **Ventaja:** Mapping automático, desacoplamiento de entidades, compilado en tiempo de compilación

### ✅ 7. **Logger Service (Abstraction)**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Helpers/Logging/ILoggerService.cs`
- **Métodos:** 
  - LogInformation - Información general
  - LogWarning - Advertencias
  - LogError - Errores con Exception
  - LogDebug - Debug information
- **Ventaja:** Cambiar proveedor de logging sin tocar código de negocio

### ✅ 8. **Global Exception Handling Middleware**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Middleware/ExceptionHandlingMiddleware.cs`
- **Función:** Captura TODAS las excepciones no manejadas
- **Respuesta:** ErrorResponse con Message, Code, StatusCode, Errors, Timestamp
- **Mapeo:**
  - ValidationException → 400
  - UnauthorizedException → 401
  - NotFoundException → 404
  - ConflictException → 409
  - AppException → statusCode personalizado
  - Otras excepciones → 500
- **Ventaja:** Manejo consistente de errores, logging automático, respuestas JSON estándar

### ✅ 9. **Validation Extensions**
- **Archivo:** `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Middleware/ValidationExtensions.cs`
- **Función:** Extensión para validar DTOs con FluentValidation
- **Método:** `ValidateAsync<T>(this T model, IValidator<T> validator)`
- **Utilidad:** Se puede llamar en servicios/controllers antes de procesar

---

## 📊 Principios SOLID Implementados

| Principio | Implementación | Beneficio |
|-----------|---|---|
| **S** - Single Responsibility | AuthService (auth), FormService (forms), ExportService (export), ValidationHelper (validaciones), StringHelper (strings) | Fácil de testear, mantener y entender |
| **O** - Open/Closed | IRepository<T> extensible, nuevos validadores sin modificar existentes | Fácil agregar nuevas funcionalidades |
| **L** - Liskov Substitution | Todas las excepciones heredan de AppException correctamente | Polymorfismo confiable |
| **I** - Interface Segregation | IAuthService, IFormService, IExportService, ILoggerService separadas | Interfaces claras y específicas |
| **D** - Dependency Inversion | Inyección de interfaces (IAuthService, IUnitOfWork) no de clases concretas | Bajo acoplamiento, testing fácil |

---

## 🔧 Configuración en Program.cs

```csharp
// 1. AutoMapper - Registra el perfil de mapeo
builder.Services.AddAutoMapper(typeof(MappingProfile));

// 2. FluentValidation - Registro automático de validadores
var assembly = typeof(Program).Assembly;
var validatorType = typeof(IValidator<>);
var validatorTypes = assembly.GetTypes()
    .Where(t => t.GetInterfaces().Any(i =>
        i.IsGenericType &&
        i.GetGenericTypeDefinition() == validatorType))
    .ToList();

foreach (var type in validatorTypes)
{
    var interfaces = type.GetInterfaces()
        .Where(i => i.IsGenericType && i.GetGenericTypeDefinition() == validatorType);
    
    foreach (var interfaceType in interfaces)
    {
        builder.Services.AddScoped(interfaceType, type);
    }
}

// 3. Unit of Work Pattern
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

// 4. Business Services
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<IFormService, FormService>();
builder.Services.AddScoped<IExportService, ExportService>();

// 5. Logger Service
builder.Services.AddScoped<ILoggerService, LoggerService>();

// 6. Middleware de excepciones (envuelve TODAS las solicitudes)
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

---

## 📦 Paquetes NuGet Instalados

```
FluentValidation                                    12.1.1
FluentValidation.DependencyInjectionExtensions     12.1.1
AutoMapper.Extensions.Microsoft.DependencyInjection 12.0.1
BCrypt.Net-Next                                    4.0.3
ClosedXML                                          0.104.1
Microsoft.AspNetCore.Authentication.JwtBearer     10.0.0
```

---

## 🚀 Mejoras en el Flujo de Solicitud

### Antes (Sin Patrones - Código Frágil)
```
Request 
  ↓
Controller (sin validación clara)
  ↓
Service (validar datos, try-catch disperso)
  ↓
Excepciones inconsistentes
  ↓
Respuestas HTTP variadas
```

### Ahora (Con Patrones - Código Robusto)
```
Request 
  ↓
ExceptionHandlingMiddleware (envoltura global)
  ↓
Controller recibe DTO
  ↓
FluentValidator valida automáticamente (si inválido → ValidationException)
  ↓
IUnitOfWork coordina transacciones
  ↓
IRepository<T> (acceso a datos)
  ↓
Entity Framework Core → SQLite
  ↓
AutoMapper transforma resultado
  ↓
Response estándar JSON (Success/Failure)
  ↓
Si error en cualquier paso → ExceptionHandlingMiddleware captura
  ↓
ErrorResponse JSON con code, message, statusCode, errors, timestamp
```

---

## ✨ Beneficios Logrados

### 🎯 Arquitectura
- ✅ Layered Architecture limpia (Entity/Data/Business/Web/Helpers)
- ✅ Separación clara de responsabilidades
- ✅ Cada capa tiene propósito específico
- ✅ Fácil de escalar y mantener
- ✅ Patrón conocido por developers

### 🧪 Testing
- ✅ Servicios inyectables y mockeable
- ✅ Excepciones tipadas y controladas
- ✅ Lógica desacoplada de frameworks
- ✅ Fácil crear tests unitarios
- ✅ Validadores reutilizables

### 📝 Código
- ✅ Más limpio y legible
- ✅ Reutilizable entre funcionalidades
- ✅ Autodocumentado con interfaces
- ✅ Menos duplicación
- ✅ Guía clara para nuevos developers

### 🛡️ Confiabilidad
- ✅ Validaciones en múltiples capas
- ✅ Manejo robusto de errores
- ✅ Transacciones atómicas (Commit/Rollback)
- ✅ Logging automático
- ✅ Respuestas consistentes

### 📊 Rendimiento
- ✅ Result Pattern sin overhead de excepciones
- ✅ Lazy loading en Unit of Work
- ✅ Mapping compilado con AutoMapper
- ✅ Queries optimizadas con Repository
- ✅ Caching a nivel de base de datos

### 🔒 Seguridad
- ✅ Validaciones contra inyección de datos
- ✅ Contraseñas hasheadas con BCrypt
- ✅ JWT tokens con firma
- ✅ Errores no revelan información sensible
- ✅ Autorización en múltiples niveles

---

## 📂 Estructura de Carpetas Implementada

```
AutoCheckAML.Api/
├── Entity/                              # Capa 1: Modelos de dominio (POO puro)
│   ├── User.cs                          # Propiedades: Id, Username, Email, PasswordHash, etc.
│   └── FormSubmission.cs                # Propiedades: Id, UserId, Nombre, Email, etc.
│
├── Data/                                # Capa 2: Acceso a datos
│   ├── AutoCheckAMLContext.cs           # DbContext con DbSets y configuraciones
│   ├── Repository/                      # Patrón Repository
│   │   ├── IRepository.cs               # Interface genérica
│   │   └── Repository.cs                # Implementación genérica
│   └── UnitOfWork/                      # Patrón Unit of Work
│       ├── IUnitOfWork.cs               # Interface con Users, FormSubmissions
│       └── UnitOfWork.cs                # Implementación con transacciones
│
├── Business/                            # Capa 3: Lógica de negocio
│   ├── IAuthService.cs                  # Interface de autenticación
│   ├── AuthService.cs                   # Autenticación y registro
│   ├── IFormService.cs                  # Interface de formularios
│   ├── FormService.cs                   # CRUD y búsqueda de formularios
│   ├── IExportService.cs                # Interface de exportación
│   └── ExportService.cs                 # Exportación a Excel
│
├── Web/                                 # Capa 4: Presentación (API)
│   ├── Controllers/                     # Endpoints HTTP
│   │   ├── AuthController.cs            # POST: login, register
│   │   └── FormSubmissionsController.cs # CRUD + search + export
│   ├── DTOs/                            # Contratos de solicitud/respuesta
│   │   ├── AuthDTOs.cs                  # LoginRequest, RegisterRequest, etc.
│   │   └── FormDTOs.cs                  # FormSubmissionRequest, etc.
│   ├── Validators/                      # Validación declarativa
│   │   └── RequestValidators.cs         # Todos los validadores
│   ├── Mapping/                         # AutoMapper
│   │   └── MappingProfile.cs            # Configuración de mapeos
│   └── Middleware/                      # Middleware custom
│       ├── ExceptionHandlingMiddleware.cs  # Manejo global de errores
│       └── ValidationExtensions.cs      # Extensiones de validación
│
├── Helpers/                             # Capa 5: Utilidades reutilizables
│   ├── Exceptions/                      # Custom Exceptions
│   │   └── AppException.cs              # Jerarquía: AppException (base) → NotFoundException, etc.
│   ├── Results/                         # Result Pattern
│   │   └── Result.cs                    # Result<T> y Result (factory methods)
│   ├── Logging/                         # Logging abstracto
│   │   └── ILoggerService.cs            # ILoggerService + LoggerService
│   ├── ValidationHelper.cs              # IsValidEmail, IsNotEmpty, IsValidPhone
│   └── StringHelper.cs                  # Normalize, Capitalize, Truncate, RemoveSpecialCharacters
│
├── Properties/                          
│   └── launchSettings.json              # Puerto 5280 para desarrollo
│
└── Program.cs                           # Punto de entrada: Configuración DI, Middleware, Database
```

---

## 🎓 Aprendizajes Clave de POO

### 1. Encapsulación
```csharp
public class User
{
    public int Id { get; set; }                    // Propiedad pública
    public string Username { get; set; }           // Controlado por EF Core
    private string _internalData;                  // Campo privado (si existiera)
}
```
- **Ventaja:** Control sobre qué se accede

### 2. Abstracción
```csharp
public interface IAuthService                     // Abstrae la implementación
{
    Task<LoginResponse> LoginAsync(LoginRequest request);
}

public class AuthService : IAuthService           // Implementa la interfaz
{
    // Detalles internos ocultos
}
```
- **Ventaja:** Se usa la interfaz, no la clase concreta

### 3. Herencia
```csharp
public class AppException : Exception             // Base
{
    public int StatusCode { get; set; }
    public string Code { get; set; }
}

public class NotFoundException : AppException     // Derivada
{
    public NotFoundException(string message) 
        : base(message, "NOT_FOUND", 404) { }
}
```
- **Ventaja:** Código reutilizable, comportamiento común

### 4. Polimorfismo
```csharp
// Mismo tipo, diferentes implementaciones
IRepository<User> userRepo = new Repository<User>(context);
IRepository<FormSubmission> formRepo = new Repository<FormSubmission>(context);

// Mismo método, diferentes validadores
IValidator<LoginRequest> loginValidator = new LoginRequestValidator();
IValidator<FormSubmissionRequest> formValidator = new FormSubmissionRequestValidator();
```
- **Ventaja:** Código genérico y flexible

---

## 🔍 Ejemplo de Flujo Completo - Autenticación

```csharp
// USUARIO HACE SOLICITUD
POST http://localhost:5280/api/auth/login
Content-Type: application/json
{
  "username": "admin",
  "password": "admin123"
}

// FLUJO INTERNO
1. ExceptionHandlingMiddleware recibe request
   ↓
2. AuthController.Login([FromBody] LoginRequest request) ejecuta
   ↓
3. FluentValidator (LoginRequestValidator) valida:
   - username: no vacío, 3-50 chars ✅
   - password: no vacío, 6+ chars ✅
   Si falla → throw ValidationException (400)
   ↓
4. IAuthService.LoginAsync(request) ejecuta:
   ├─ IUnitOfWork.Users.FirstOrDefaultAsync(u => u.Username == "admin")
   │  └─ Repository busca en BD
   │  └─ Si no existe → throw NotFoundException (404)
   ├─ BCrypt.Verify(request.Password, user.PasswordHash)
   │  └─ Si falla → throw UnauthorizedException (401)
   ├─ GenerateJwtToken(user)
   │  └─ Firma token con secret
   ├─ user.LastLogin = DateTime.Now
   └─ await _unitOfWork.SaveChangesAsync()
   ↓
5. AutoMapper.Map<LoginResponse>(user)
   └─ Transforma User → LoginResponse (excluye PasswordHash)
   ↓
6. Asigna token a response.Token
   ↓
7. return Ok(response);  // 200 OK

// RESPUESTA AL USUARIO
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": 1,
  "username": "admin",
  "email": "admin@autocheck.com",
  "fullName": "Administrador",
  "token": "eyJhbGc..."  // JWT Token
}
```

---

## 📊 Matriz de Patrones vs Problemas Resueltos

| Problema | Patrón | Solución | Beneficio |
|----------|--------|----------|-----------|
| Acceso a datos acoplado | Repository | IRepository<T> abstrae DbContext | Cambiar BD sin afectar servicios |
| Transacciones inconsistentes | Unit of Work | Coordina múltiples repos | Rollback automático |
| Excepciones dispersas | Custom Exceptions | AppException + tipos específicos | Manejo consistente |
| Validación manual | FluentValidation | Validadores declarativos | Menos código, más claro |
| Mapping propenso a errores | AutoMapper | Mapping automático compilado | Menos bugs |
| Errores inconsistentes | Global Middleware | ErrorResponse estándar | Respuestas predecibles |
| Logging acoplado | Logger Service | ILoggerService abstracto | Cambiar proveedor fácil |
| Dependencias hardcodeadas | DI Container | Inyección automática | Testing y escalabilidad |

---

**Estado:** ✅ Compilación exitosa, todos los patrones implementados
**Base de datos:** SQLite con usuario admin/admin123
**Servidor:** Escuchando en http://localhost:5280

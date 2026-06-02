# Patrones de Diseño y Buenas Prácticas Implementados

## 🎯 Resumen de Patrones

### 1. **Repository Pattern**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Data/Repository/IRepository.cs`

**Propósito:** Abstrae el acceso a datos y centraliza las operaciones CRUD.

**Beneficios:**
- ✅ Desacopla la lógica de negocio del acceso a datos
- ✅ Facilita testing (se puede mockear fácilmente)
- ✅ Código reutilizable para cualquier entidad

**Ejemplo de uso:**
```csharp
var user = await _unitOfWork.Users.FirstOrDefaultAsync(u => u.Username == username);
```

---

### 2. **Unit of Work Pattern**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Data/UnitOfWork/IUnitOfWork.cs`

**Propósito:** Coordina múltiples repositorios en una única transacción.

**Beneficios:**
- ✅ Manejo consistente de transacciones
- ✅ Commit/Rollback atómicos
- ✅ Evita inconsistencias de datos

**Ejemplo de uso:**
```csharp
await _unitOfWork.BeginTransactionAsync();
await _unitOfWork.FormSubmissions.AddAsync(form);
await _unitOfWork.CommitTransactionAsync();
```

---

### 3. **Dependency Injection**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Program.cs`

**Propósito:** Inyecta dependencias automáticamente mediante contenedor IoC.

**Beneficios:**
- ✅ Código más testeable
- ✅ Desacoplamiento de clases
- ✅ Configuración centralizada

**Configuración en Program.cs:**
```csharp
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<ILoggerService, LoggerService>();
```

---

### 4. **Exception Handling Pattern**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Helpers/Exceptions/AppException.cs` + `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Middleware/ExceptionHandlingMiddleware.cs`

**Propósito:** Excepciones personalizadas y middleware global de manejo de errores.

**Beneficios:**
- ✅ Errores consistentes en toda la aplicación
- ✅ Respuestas HTTP estándar
- ✅ Logging automático de excepciones

**Excepciones disponibles:**
```csharp
- AppException         // Excepción base (500)
- NotFoundException    // 404
- ValidationException  // 400 (con diccionario de errores)
- UnauthorizedException // 401
- ConflictException    // 409
```

---

### 5. **Result Pattern**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Helpers/Results/Result.cs`

**Propósito:** Alternativa funcional a excepciones para operaciones que pueden fallar.

**Beneficios:**
- ✅ Sin overhead de excepciones
- ✅ Resultados explícitos (éxito/error)
- ✅ Mejor rendimiento en rutas de error

**Ejemplo de uso:**
```csharp
var result = Result<User>.Success(user, "Usuario autenticado correctamente");
var failure = Result<User>.Failure("Usuario no encontrado", "USER_NOT_FOUND");
```

---

### 6. **Fluent Validation**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Validators/RequestValidators.cs`

**Propósito:** Validación declarativa y reutilizable de DTOs.

**Beneficios:**
- ✅ Código limpio y legible
- ✅ Validaciones complejas fáciles
- ✅ Reutilizable en múltiples capas

**Validadores disponibles:**
- `LoginRequestValidator`
- `RegisterRequestValidator`
- `FormSubmissionRequestValidator`
- `FormFilterRequestValidator`
- `StatusUpdateRequestValidator`

**Ejemplo:**
```csharp
public class LoginRequestValidator : AbstractValidator<LoginRequest>
{
    public LoginRequestValidator()
    {
        RuleFor(x => x.Username)
            .NotEmpty().WithMessage("El usuario es requerido")
            .MinimumLength(3).WithMessage("Mínimo 3 caracteres");
    }
}
```

---

### 7. **AutoMapper (Object Mapping)**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Web/Mapping/MappingProfile.cs`

**Propósito:** Mapeo automático entre entidades y DTOs.

**Beneficios:**
- ✅ Evita mapping manual propenso a errores
- ✅ Desacopla entidades de DTOs
- ✅ Configuración centralizada

**Mapeos configurados:**
```csharp
- User → LoginResponse
- User → RegisterResponse
- FormSubmissionRequest → FormSubmission
- FormSubmission → FormSubmissionResponse
```

**Ejemplo:**
```csharp
CreateMap<User, LoginResponse>()
    .ForMember(dest => dest.Token, opt => opt.Ignore());

var response = _mapper.Map<LoginResponse>(user);
```

---

### 8. **Logger Service (Abstraction)**
**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Helpers/Logging/ILoggerService.cs`

**Propósito:** Abstrae la implementación de logging.

**Beneficios:**
- ✅ Cambiar proveedor de logging sin tocar código de negocio
- ✅ Métodos de conveniencia
- ✅ Logging consistente

**Métodos disponibles:**
```csharp
void LogInformation(string message, params object[] args);
void LogWarning(string message, params object[] args);
void LogError(Exception ex, string message, params object[] args);
void LogDebug(string message, params object[] args);
```

**Ejemplo:**
```csharp
_logger.LogInformation("Usuario {Username} autenticado", username);
_logger.LogError(ex, "Error al procesar formulario");
```

---

## 🏗️ Principios SOLID Implementados

### **S - Single Responsibility Principle**
Cada clase tiene una responsabilidad única:
- `AuthService` → Autenticación
- `FormService` → Gestión de formularios
- `ExportService` → Exportación a Excel
- `ValidationHelper` → Validaciones comunes
- `StringHelper` → Operaciones con strings

### **O - Open/Closed Principle**
Abierto para extensión, cerrado para modificación:
- `IRepository<T>` permite agregar nuevos repositorios específicos
- Validadores pueden extenderse sin modificar los existentes
- Excepciones se pueden crear derivadas de `AppException`

### **L - Liskov Substitution Principle**
Subclases pueden reemplazar clases base:
- Toda excepción personalizada hereda de `AppException`
- Todos los servicios implementan sus interfaces
- Todas las implementaciones de `IRepository<T>` son intercambiables

### **I - Interface Segregation Principle**
Interfaces pequeñas y específicas:
- `IRepository<T>` para operaciones genéricas
- `IUnitOfWork` para coordinar múltiples repositorios
- `IAuthService`, `IFormService`, `IExportService` para servicios específicos
- `ILoggerService` para logging

### **D - Dependency Inversion Principle**
Dependencias en abstracciones, no en concreciones:
```csharp
public class AuthController
{
    private readonly IAuthService _authService;  // Inyecta interfaz, no clase
    
    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }
}
```

---

## 📁 Estructura de Carpetas (Arquitectura en Capas)

```
BackEnd-AutoCheck/AutoCheckAML.Api/
├── Entity/                          # Capa de entidades (POO)
│   ├── User.cs                      # Modelo de usuario
│   └── FormSubmission.cs            # Modelo de formulario
│
├── Data/                            # Capa de datos
│   ├── AutoCheckAMLContext.cs       # DbContext (EF Core)
│   ├── Repository/                  # Repository Pattern
│   │   ├── IRepository.cs           # Interface genérica
│   │   └── Repository.cs            # Implementación genérica
│   └── UnitOfWork/                  # Unit of Work Pattern
│       ├── IUnitOfWork.cs           # Interface
│       └── UnitOfWork.cs            # Implementación
│
├── Business/                        # Capa de negocio
│   ├── AuthService.cs               # Autenticación
│   ├── IAuthService.cs              # Interface
│   ├── FormService.cs               # Gestión de formularios
│   ├── IFormService.cs              # Interface
│   ├── ExportService.cs             # Exportación Excel
│   ├── IExportService.cs            # Interface
│   └── (Aquí van todos los servicios)
│
├── Web/                             # Capa de presentación
│   ├── Controllers/                 # API Endpoints
│   │   ├── AuthController.cs        # POST login/register
│   │   └── FormSubmissionsController.cs  # CRUD de formularios
│   ├── DTOs/                        # Data Transfer Objects
│   │   ├── AuthDTOs.cs              # Login, Register requests/responses
│   │   └── FormDTOs.cs              # Form requests/responses
│   ├── Validators/                  # FluentValidation
│   │   └── RequestValidators.cs     # Validadores de DTOs
│   ├── Mapping/                     # AutoMapper
│   │   └── MappingProfile.cs        # Perfiles de mapeo
│   └── Middleware/                  # Middleware custom
│       ├── ExceptionHandlingMiddleware.cs   # Manejo global de errores
│       └── ValidationExtensions.cs         # Extensiones de validación
│
├── Helpers/                         # Utilidades reutilizables
│   ├── Exceptions/                  # Custom Exceptions
│   │   └── AppException.cs          # Jerarquía de excepciones
│   ├── Results/                     # Result Pattern
│   │   └── Result.cs                # Result<T> y Result
│   ├── Logging/                     # Logging abstracto
│   │   └── ILoggerService.cs        # Logger Service
│   ├── ValidationHelper.cs          # Validaciones comunes
│   └── StringHelper.cs              # Operaciones con strings
│
└── Program.cs                       # Punto de entrada y configuración DI
```

---

## 🔄 Flujo de Solicitud con Patrones

### Ejemplo: Autenticación de Usuario (POST /api/auth/login)

```
1. HTTP Request arrives at AuthController
   ↓
2. ExceptionHandlingMiddleware envuelve la solicitud
   ↓
3. AuthController.Login(LoginRequest request) receives DTO
   ↓
4. FluentValidator valida automáticamente LoginRequest
   ├─ Si es inválido → ValidationException
   │  └─ Middleware captura y retorna 400 Bad Request
   └─ Si es válido → Continúa
   ↓
5. IAuthService.LoginAsync(request) ejecuta
   ├─ IUnitOfWork.Users.FirstOrDefaultAsync() busca usuario
   │  └─ Repository<User> → EF Core → SQLite database
   ├─ BCrypt.Verify() valida contraseña
   ├─ GenerateJwtToken(user) crea token
   └─ UpdateLastLogin() registra acceso
   ↓
6. AutoMapper mapea User → LoginResponse
   ├─ Se asigna el JWT token
   └─ Se obtiene respuesta tipada
   ↓
7. Controller retorna 200 OK con LoginResponse
   └─ Body: { Id, Username, Email, FullName, Token }

Si hay error en cualquier paso:
└─ Exception capturada por ExceptionHandlingMiddleware
   └─ Retorna ErrorResponse con:
      ├─ Message: descripción del error
      ├─ Code: código único del error
      ├─ StatusCode: código HTTP (400/401/404/500)
      ├─ Errors: diccionario de errores por campo (si aplica)
      └─ Timestamp: cuándo ocurrió
```

---

## 🚀 Ventajas de Esta Arquitectura

| Aspecto | Ventaja | Patrón |
|--------|---------|--------|
| **Mantenibilidad** | Código organizado y predecible | Layered Architecture |
| **Escalabilidad** | Fácil agregar nuevas funcionalidades | Repository + Unit of Work |
| **Testabilidad** | Mock de dependencias trivial | DI + Interfaces |
| **Reutilización** | Componentes pueden usarse en múltiples contextos | DI Container |
| **Performance** | Result Pattern sin overhead de excepciones | Result Pattern |
| **Seguridad** | Validaciones en múltiples capas | FluentValidation |
| **Documentación** | Código autoexplicativo con interfaces claras | SOLID + Naming |
| **Robustez** | Manejo consistente de errores | Exception Handling |

---

## 📊 Paquetes NuGet Utilizados

```
FluentValidation                                    12.1.1
FluentValidation.DependencyInjectionExtensions     12.1.1
AutoMapper.Extensions.Microsoft.DependencyInjection 12.0.1
BCrypt.Net-Next                                    4.0.3
ClosedXML                                          0.104.1
```

---

## 📝 Próximos Pasos (Opcionales)

- [ ] Specification Pattern (filtros complejos)
- [ ] Caching Pattern
- [ ] Audit Log Pattern
- [ ] Pagination Helper reutilizable
- [ ] API Versioning
- [ ] Rate Limiting
- [ ] Health Checks
- [ ] Swagger mejorado con ejemplos

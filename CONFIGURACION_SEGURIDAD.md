# Configuración de Seguridad - AutoCheckAML

## Variables de Entorno Requeridas

Para ejecutar la aplicación de forma segura, debes configurar las siguientes variables de entorno:

### Backend (.NET API)

#### Variables Críticas de Seguridad

```bash
# JWT Secret Key (mínimo 32 caracteres)
JWT_SECRET="tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui"

# Contraseñas por defecto para usuarios de prueba
ADMIN_DEFAULT_PASSWORD="tu-contraseña-admin-segura"
INGENIERO_DEFAULT_PASSWORD="tu-contraseña-ingeniero-segura"
SUPERVISOR_HSEQ_DEFAULT_PASSWORD="tu-contraseña-supervisor-hseq-segura"
CUADRILLA_DEFAULT_PASSWORD="tu-contraseña-cuadrilla-segura"

# Orígenes permitidos para CORS (separados por comas)
ALLOWED_ORIGINS="http://localhost:3000,http://localhost:5173,https://tu-dominio.com"
```

#### Cadena de Conexión a Base de Datos

```bash
# PostgreSQL Connection String
ConnectionStrings__DefaultConnection="Host=localhost;Database=autocheckaml;Username=postgres;Password=tu-password-seguro"
```

### Configuración en Windows

#### PowerShell (Temporal - solo para la sesión actual)
```powershell
$env:JWT_SECRET="tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui"
$env:ADMIN_DEFAULT_PASSWORD="Admin2026"
$env:INGENIERO_DEFAULT_PASSWORD="Ingeniero2026"
$env:SUPERVISOR_HSEQ_DEFAULT_PASSWORD="SupervisorHSEQ2026"
$env:CUADRILLA_DEFAULT_PASSWORD="Cuadrilla2026"
$env:ALLOWED_ORIGINS="http://localhost:3000,http://localhost:5173"
```

#### PowerShell (Permanente - usuario actual)
```powershell
[System.Environment]::SetEnvironmentVariable('JWT_SECRET', 'tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui', 'User')
[System.Environment]::SetEnvironmentVariable('ADMIN_DEFAULT_PASSWORD', 'Admin2026', 'User')
[System.Environment]::SetEnvironmentVariable('INGENIERO_DEFAULT_PASSWORD', 'Ingeniero2026', 'User')
[System.Environment]::SetEnvironmentVariable('SUPERVISOR_HSEQ_DEFAULT_PASSWORD', 'SupervisorHSEQ2026', 'User')
[System.Environment]::SetEnvironmentVariable('CUADRILLA_DEFAULT_PASSWORD', 'Cuadrilla2026', 'User')
[System.Environment]::SetEnvironmentVariable('ALLOWED_ORIGINS', 'http://localhost:3000,http://localhost:5173', 'User')
```

#### CMD (Temporal)
```cmd
set JWT_SECRET=tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui
set ADMIN_DEFAULT_PASSWORD=Admin2026
set INGENIERO_DEFAULT_PASSWORD=Ingeniero2026
set SUPERVISOR_HSEQ_DEFAULT_PASSWORD=SupervisorHSEQ2026
set CUADRILLA_DEFAULT_PASSWORD=Cuadrilla2026
set ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173
```

### Configuración en Linux/macOS

```bash
# Añadir al archivo ~/.bashrc o ~/.zshrc
export JWT_SECRET="tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui"
export ADMIN_DEFAULT_PASSWORD="Admin2026"
export INGENIERO_DEFAULT_PASSWORD="Ingeniero2026"
export SUPERVISOR_HSEQ_DEFAULT_PASSWORD="SupervisorHSEQ2026"
export CUADRILLA_DEFAULT_PASSWORD="Cuadrilla2026"
export ALLOWED_ORIGINS="http://localhost:3000,http://localhost:5173"

# Luego ejecutar
source ~/.bashrc  # o source ~/.zshrc
```

### Configuración con Docker Compose

Edita el archivo `docker-compose.yml` o `docker-compose.dev.yml`:

```yaml
services:
  api:
    environment:
      - JWT_SECRET=tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui
      - ADMIN_DEFAULT_PASSWORD=Admin2026
      - INGENIERO_DEFAULT_PASSWORD=Ingeniero2026
      - SUPERVISOR_HSEQ_DEFAULT_PASSWORD=SupervisorHSEQ2026
      - CUADRILLA_DEFAULT_PASSWORD=Cuadrilla2026
      - ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173
      - ConnectionStrings__DefaultConnection=Host=db;Database=autocheckaml;Username=postgres;Password=tu-password-seguro
```

O mejor aún, usa un archivo `.env`:

```bash
# Crear archivo .env en la raíz del proyecto
JWT_SECRET=tu-clave-secreta-jwt-de-minimo-32-caracteres-aqui
ADMIN_DEFAULT_PASSWORD=Admin2026
INGENIERO_DEFAULT_PASSWORD=Ingeniero2026
SUPERVISOR_HSEQ_DEFAULT_PASSWORD=SupervisorHSEQ2026
CUADRILLA_DEFAULT_PASSWORD=Cuadrilla2026
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173
DB_PASSWORD=tu-password-seguro
```

Y en `docker-compose.yml`:

```yaml
services:
  api:
    env_file:
      - .env
    environment:
      - ConnectionStrings__DefaultConnection=Host=db;Database=autocheckaml;Username=postgres;Password=${DB_PASSWORD}
```

## Valores por Defecto en appsettings.json

Si NO se configuran las variables de entorno, el sistema usará los valores de `appsettings.json`. Sin embargo, estos valores **DEBEN ser cambiados antes de producción** ya que tienen marcadores como `CHANGE-THIS`.

### ⚠️ IMPORTANTE para Producción

1. **NUNCA** commits las variables de entorno reales al repositorio Git
2. Usa servicios de gestión de secretos en producción:
   - Azure Key Vault
   - AWS Secrets Manager
   - HashiCorp Vault
   - Variables de entorno del sistema/contenedor

3. Asegúrate de que `.env` esté en `.gitignore`

4. Genera claves JWT seguras:
   ```bash
   # Ejemplo en PowerShell para generar una clave aleatoria
   -join ((65..90) + (97..122) + (48..57) | Get-Random -Count 32 | % {[char]$_})
   ```

## Verificación de Configuración

Al iniciar la aplicación, el sistema validará:

1. ✅ JWT Secret tiene al menos 32 caracteres
2. ✅ Contraseñas de usuarios por defecto están configuradas
3. ✅ Orígenes CORS están definidos

Si falta alguna configuración, la aplicación mostrará un error claro indicando qué variable falta.

## Cambios de Seguridad Realizados

### Arreglados en esta actualización:

1. ✅ **Credenciales hardcodeadas removidas** - Ahora usan variables de entorno
2. ✅ **JWT Secret configurable** - Con validación de longitud mínima
3. ✅ **Fallback de contraseñas en texto plano eliminado** - Solo BCrypt
4. ✅ **CORS restrictivo** - Ya no permite todos los orígenes
5. ✅ **HTTPS enforced en producción** - RequireHttpsMetadata activado para producción
6. ✅ **Sistema de permisos corregido** - Roles SOFTWARE pueden asignar permisos
7. ✅ **RoleDto corregido** - Ahora envía objetos de permisos completos con IDs

## Contacto

Para soporte técnico relacionado con la configuración de seguridad, contactar al equipo de desarrollo.

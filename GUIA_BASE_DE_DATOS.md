# Guía de Base de Datos PostgreSQL - AutoCheckAML

## ✅ Estado Actual: PostgreSQL Configurado

El proyecto **ya está migrado completamente a PostgreSQL**. SQLite ha sido eliminado.

---

## 🚀 Opciones para Ejecutar la Aplicación

### Opción 1: Con Docker (Recomendado) 🐳

#### Desarrollo
```bash
# Iniciar todo (PostgreSQL + Backend + Frontend)
docker-compose -f docker-compose.dev.yml up

# O solo la base de datos
docker-compose -f docker-compose.dev.yml up db
```

#### Producción
```bash
docker-compose up
```

**Ventajas:**
- ✅ Todo configurado automáticamente
- ✅ No necesitas instalar PostgreSQL localmente
- ✅ Ambiente aislado y reproducible

**URLs:**
- Backend: http://localhost:5000/api
- Frontend: http://localhost:5173 (dev) o http://localhost:3000 (prod)
- PostgreSQL: localhost:5439 (mapeado desde el puerto 5432 del contenedor)

---

### Opción 2: Sin Docker (Desarrollo Local)

Si quieres ejecutar el backend localmente con Visual Studio/VS Code, necesitas PostgreSQL instalado.

#### Paso 1: Instalar PostgreSQL en Windows

**Opción A: Instalador Oficial**
1. Descargar de: https://www.postgresql.org/download/windows/
2. Ejecutar instalador
3. Configurar:
   - Puerto: `5432`
   - Usuario: `postgres`
   - Contraseña: `postgrespassword` (o la que prefieras)
4. Crear base de datos:
   ```sql
   CREATE DATABASE autocheckaml;
   ```

**Opción B: Docker solo para PostgreSQL**
```bash
# Solo ejecutar PostgreSQL en Docker
docker run -d \
  --name postgres-autocheckaml \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgrespassword \
  -e POSTGRES_DB=autocheckaml \
  -p 5432:5432 \
  postgres:15-alpine
```

**Opción C: Chocolatey** (si tienes Chocolatey instalado)
```powershell
choco install postgresql
```

#### Paso 2: Configurar Variables de Entorno

```powershell
# Variables requeridas
$env:JWT_SECRET="development-secret-key-minimum-32-characters-for-local"
$env:ADMIN_DEFAULT_PASSWORD="Admin2026"
$env:INGENIERO_DEFAULT_PASSWORD="Ingeniero2026"
$env:SUPERVISOR_HSEQ_DEFAULT_PASSWORD="SupervisorHSEQ2026"
$env:CUADRILLA_DEFAULT_PASSWORD="Cuadrilla2026"
$env:ALLOWED_ORIGINS="http://localhost:3000,http://localhost:5173"

# Si cambiaste la contraseña de PostgreSQL
$env:ConnectionStrings__DefaultConnection="Host=localhost;Database=autocheckaml;Username=postgres;Password=TU_PASSWORD"
```

#### Paso 3: Ejecutar el Backend

```bash
cd BackEnd-AutoCheck/AutoCheckAML.Api
dotnet restore
dotnet run
```

El backend iniciará en: http://localhost:5000

#### Paso 4: Ejecutar el Frontend

```bash
cd FrontEnd-AutoCheck
npm install
npm run dev
```

El frontend iniciará en: http://localhost:5173

---

## 🗄️ Conexiones a la Base de Datos

### Con Docker
- **Host**: `localhost` (desde tu máquina) o `db` (desde otro contenedor)
- **Puerto**: `5439` (mapeado desde el contenedor)
- **Usuario**: `postgres`
- **Contraseña**: `postgrespassword`
- **Base de datos**: `autocheckaml`

**Cadena de conexión:**
```
Host=localhost;Port=5439;Database=autocheckaml;Username=postgres;Password=postgrespassword
```

### Sin Docker (Local)
- **Host**: `localhost`
- **Puerto**: `5432`
- **Usuario**: `postgres`
- **Contraseña**: `postgrespassword` (o la que configuraste)
- **Base de datos**: `autocheckaml`

**Cadena de conexión:**
```
Host=localhost;Database=autocheckaml;Username=postgres;Password=postgrespassword
```

---

## 🔧 Herramientas de Gestión de PostgreSQL

### pgAdmin (GUI - Recomendado para principiantes)
1. Descargar: https://www.pgadmin.org/download/
2. Instalar y abrir
3. Conectar con las credenciales de arriba

### DBeaver (GUI - Multiplataforma)
1. Descargar: https://dbeaver.io/download/
2. Soporta múltiples bases de datos

### Azure Data Studio
1. Descargar: https://docs.microsoft.com/azure-data-studio/download
2. Instalar extensión de PostgreSQL

### psql (CLI - Avanzado)
```bash
# Conectar a PostgreSQL
psql -h localhost -p 5432 -U postgres -d autocheckaml

# Comandos útiles
\dt                  # Listar tablas
\d "Users"           # Describir tabla Users
SELECT * FROM "Roles";  # Query
\q                   # Salir
```

---

## 🔄 Reiniciar Base de Datos

### Con Docker
```bash
# Detener y eliminar todo (incluyendo volúmenes)
docker-compose down -v

# Iniciar de nuevo (creará BD limpia)
docker-compose up
```

### Sin Docker
```sql
-- Conectar a PostgreSQL y ejecutar:
DROP DATABASE autocheckaml;
CREATE DATABASE autocheckaml;
```

Luego reinicia el backend para que vuelva a crear las tablas.

---

## 📋 Usuarios de Prueba Creados Automáticamente

Cuando inicias la aplicación por primera vez, se crean estos usuarios:

| Usuario | Email | Contraseña | Rol |
|---------|-------|------------|-----|
| Admin | admin@autocheck.com | Admin2026 | DEV |
| ingeniero | ingeniero@autocheck.com | Ingeniero2026 | INGENIERO_MECANICO |
| supervisorhseq | supervisorhseq@autocheck.com | SupervisorHSEQ2026 | SUPERVISOR_HSEQ |
| cuadrilla | cuadrilla@autocheck.com | Cuadrilla2026 | CUADRILLA |

**Nota:** Estas contraseñas se configuran mediante variables de entorno. Ver `CONFIGURACION_SEGURIDAD.md`.

---

## 🐛 Solución de Problemas

### Error: "No se puede conectar a PostgreSQL"

**Verificar si PostgreSQL está corriendo:**
```bash
# Windows (si instalaste PostgreSQL)
Get-Service postgresql*

# Docker
docker ps | grep postgres
```

**Verificar puerto:**
```bash
netstat -an | findstr 5432
```

### Error: "Database does not exist"

El backend crea la BD automáticamente la primera vez. Si falla:
```sql
CREATE DATABASE autocheckaml;
```

### Error: "Password authentication failed"

Verifica la contraseña en la cadena de conexión:
- `appsettings.json` → línea 21
- O variable de entorno: `ConnectionStrings__DefaultConnection`

### Error: "Role SUPERVISOR_HSEQ no existe"

Reinicia la base de datos. Los roles se crean automáticamente en `AutoCheckAMLContext.cs`.

### Error al ejecutar localmente: "SQLite provider not found"

✅ **Ya solucionado**. El paquete SQLite ha sido eliminado del proyecto.

---

## 📊 Estructura de la Base de Datos

Las tablas se crean automáticamente usando Entity Framework Core:

### Tablas Principales
- **Users** - Usuarios del sistema
- **Roles** - Roles (DEV, SOFTWARE, INGENIERO_MECANICO, SUPERVISOR_HSEQ, CUADRILLA)
- **Permissions** - Permisos granulares
- **UserRoles** - Relación usuarios-roles (N:M)
- **RolePermissions** - Relación roles-permisos (N:M)
- **Crews** - Cuadrillas de trabajo
- **FormTemplates** - Plantillas de formularios
- **FormFields** - Campos de formularios
- **FormSubmissions** - Formularios completados
- **Answers** - Respuestas a campos
- **Attachments** - Archivos adjuntos
- **AuditLog** - Registro de auditoría
- **ExportHistory** - Historial de exportaciones
- **RefreshTokens** - Tokens de refresco JWT

---

## 🔐 Seguridad

**IMPORTANTE:**
- ❌ **NO** subas `appsettings.json` con contraseñas reales a Git
- ✅ **USA** variables de entorno para producción
- ✅ **CAMBIA** la contraseña de PostgreSQL en producción
- ✅ **REVISA** `CONFIGURACION_SEGURIDAD.md` para mejores prácticas

---

## 📞 Soporte

Si tienes problemas:
1. Revisa esta guía
2. Revisa `CONFIGURACION_SEGURIDAD.md`
3. Revisa `CAMBIOS_REALIZADOS.md`
4. Verifica logs del backend
5. Verifica logs de PostgreSQL

---

**Última actualización:** 2026-06-16

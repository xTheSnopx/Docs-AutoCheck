# ✅ Checklist para Exposición - AutoCheckAML

## 🎯 Estado Actual del Sistema

### ✅ Infraestructura Dockerizada
- **Base de Datos:** PostgreSQL 15 Alpine (Puerto 5439)
- **Backend API:** .NET 10.0 (Puerto 5000)
- **Frontend:** React 19 + Nginx (Puerto 3000)
- **Estado:** Todos los contenedores corriendo correctamente

---

## 📊 Datos del Sistema

### Usuarios Configurados (4 usuarios activos)

| ID | Usuario | Nombre | Rol | Contraseña Default |
|----|---------|--------|-----|-------------------|
| 1 | Admin | Administrador del Sistema | DEV | Admin2026 |
| 2 | ingeniero | Ingeniero Mecánico Juan | INGENIERO_MECANICO | (configurar) |
| 3 | supervisorhseq | Supervisor HSEQ Maria | SUPERVISOR_HSEQ | Hseq2026 |
| 4 | cuadrilla | Operador Cuadrilla Carlos | CUADRILLA | (configurar) |

**⚠️ IMPORTANTE:** Verifica las contraseñas antes de la demo. Usa este comando:

```sql
-- Ver estado de contraseñas
SELECT "Username", "FullName", LENGTH("PasswordHash") as "HashLength"
FROM "Users"
WHERE "IsDeleted" = false;

-- Si necesitas resetear alguna:
UPDATE "Users" SET "PasswordHash" = 'Admin2026' WHERE "Username" = 'Admin';
```

### Roles y Permisos (5 roles, 19 permisos)

**Roles Disponibles:**
1. **DEV** - Acceso total al sistema
2. **SOFTWARE** - Administración de usuarios, roles y permisos
3. **INGENIERO_MECANICO** - Gestión de inspecciones y reportes
4. **SUPERVISOR_HSEQ** - Supervisión y aprobación de inspecciones
5. **CUADRILLA** - Diligenciamiento de formularios

### Cuadrillas (0 configuradas)

**⚠️ PENDIENTE:** No hay cuadrillas creadas. Si quieres demostrar esta funcionalidad, créalas antes.

### Inspecciones (0 registradas)

**⚠️ PENDIENTE:** No hay inspecciones en el sistema. Para demostrar el dashboard:

**Opción 1:** Diligencia inspecciones manualmente desde la app
**Opción 2:** Inserta datos de prueba (ver sección abajo)

---

## 🚀 URLs de Acceso

```
Frontend:  http://localhost:3000
Backend:   http://localhost:5000/api
Swagger:   http://localhost:5000/swagger (si está habilitado)
DB:        localhost:5439 (postgres/AutoCheck2026)
```

---

## 📋 Flujo de Demostración Recomendado

### 1️⃣ Login y Autenticación (2 min)
1. Abrir http://localhost:3000
2. Login como **Admin** / **Admin2026**
3. Mostrar JWT token en DevTools (F12 → Application → Local Storage)
4. Explicar: "El sistema usa JWT para autenticación segura con roles y permisos"

### 2️⃣ Panel Administrativo - Gestión de Usuarios (3 min)
1. Ir a sección **"Usuarios"**
2. Mostrar lista de usuarios actuales
3. Demostrar **creación de usuario:**
   - Click en "+ Nuevo Usuario"
   - Llenar formulario
   - Asignar rol
4. Demostrar **cambio de contraseña:**
   - Click en icono de llave azul 🔑
   - Cambiar contraseña
   - Explicar: "Las contraseñas se encriptan con BCrypt automáticamente"

### 3️⃣ Gestión de Roles y Permisos (3 min)
1. Ir a sección **"Configuración"** (icono de engranaje)
2. Seleccionar un rol (ej: INGENIERO_MECANICO)
3. Mostrar matriz de permisos
4. Activar/desactivar permisos en tiempo real
5. Explicar: "El sistema tiene 19 permisos granulares que controlan el acceso"

### 4️⃣ Gestión de Cuadrillas (2 min)
**⚠️ Si hay tiempo, crear una cuadrilla antes de la demo**

1. Ir a sección **"Cuadrillas"**
2. Crear nueva cuadrilla:
   - Nombre: "Cuadrilla Zona Norte"
   - Departamento: "Mantenimiento"
   - Ubicación: "Planta Norte"
3. Asignar miembros
4. Explicar: "Las cuadrillas agrupan operadores para asignar inspecciones"

### 5️⃣ Dashboard y Métricas (3 min)
**⚠️ Si no hay datos, explica la funcionalidad teóricamente o inserta datos de prueba**

1. Ir a **"Dashboard"** (vista principal)
2. Mostrar KPIs:
   - Total de inspecciones
   - Vehículos en mantenimiento (INOP)
   - % Disponibilidad de flota
3. Mostrar tabla de inspecciones recientes
4. Usar filtros (fecha, placa, estado)
5. Explicar: "Los datos se actualizan en tiempo real desde PostgreSQL"

### 6️⃣ Formulario de Inspección (4 min)
**⚠️ CRÍTICO: Este es el core del sistema**

1. Click en "+ Nueva Inspección"
2. Mostrar formulario dinámico:
   - Campos de texto, numéricos, selección
   - Validaciones en tiempo real
   - Sección de firmas digitales
3. Diligenciar una inspección completa
4. Enviar formulario
5. Explicar: "Los formularios son configurables y se adaptan a diferentes tipos de inspección"

### 7️⃣ Flujo de Aprobación (2 min)
**⚠️ Requiere tener una inspección diligenciada**

1. Ir a **"Inspecciones"**
2. Ver detalle de una inspección
3. Aprobar/Rechazar con observaciones
4. Explicar estados:
   - PROGRAMADO (pendiente)
   - OPERATIVO (aprobado)
   - INOPERATIVO (requiere mantenimiento)

### 8️⃣ Exportación de Datos (1 min)
1. Desde cualquier tabla, click en "Exportar Excel"
2. Descargar archivo
3. Abrir en Excel y mostrar datos
4. Explicar: "Todos los módulos soportan exportación a Excel"

### 9️⃣ Auditoría y Trazabilidad (1 min)
1. Ir a **"Bitácora"** (si está visible)
2. Mostrar log de acciones:
   - Quién hizo qué
   - Cuándo
   - Desde qué IP
3. Explicar: "Todo cambio en el sistema queda registrado"

---

## 🔧 Preparación Pre-Exposición

### Checklist 30 minutos antes:

```bash
# 1. Reiniciar todos los contenedores
cd "C:\Users\XJuan\source\repos\AutoCheckAML"
docker-compose down
docker-compose up -d

# 2. Esperar 15 segundos
powershell -Command "Start-Sleep -Seconds 15"

# 3. Verificar que todo esté UP
docker ps

# 4. Verificar logs del backend (sin errores)
docker logs autocheckaml-backend --tail 30

# 5. Abrir navegador y probar login
# URL: http://localhost:3000
# Usuario: Admin
# Contraseña: Admin2026
```

### Datos de Prueba (OPCIONAL)

Si quieres tener inspecciones de ejemplo para mostrar el dashboard, ejecuta:

```sql
-- Conectar a PostgreSQL
docker exec -it autocheckaml-db psql -U postgres -d autocheckaml

-- Insertar inspección de prueba
INSERT INTO "FormSubmissions" (
    "FormTemplateId", "SubmittedByUserId", "SubmittedAt",
    "Status", "ObservationsByRespondent", "ObservationsByRectifier",
    "ActivityLocation", "RequiresReview", "IsActive", "IsDeleted", "CreatedAt"
) VALUES (
    1, 1, NOW(), 'Approved',
    'Inspección de prueba - Vehículo operativo', '',
    'Planta Norte', false, true, false, NOW()
);

-- Insertar respuestas de la inspección
-- (Ajustar según los campos de tu FormTemplate)
```

### Crear Cuadrilla de Ejemplo

```sql
INSERT INTO "Crews" (
    "Name", "Description", "Department", "Location",
    "ManagedByUserId", "IsActive", "IsDeleted", "CreatedAt"
) VALUES (
    'Cuadrilla Zona Norte',
    'Encargada del mantenimiento preventivo zona norte',
    'Mantenimiento',
    'Planta Norte',
    1,
    true,
    false,
    NOW()
);
```

---

## 🎤 Puntos Clave para Destacar

### Funcionalidades Técnicas:
✅ **Arquitectura Moderna:** React 19 + .NET 10 + PostgreSQL
✅ **Dockerizada:** Fácil despliegue y escalabilidad
✅ **Seguridad:** JWT + BCrypt + RBAC (Role-Based Access Control)
✅ **Tiempo Real:** Dashboard actualizado con datos reales
✅ **Trazabilidad:** Auditoría completa de todas las acciones
✅ **Responsive:** Funciona en desktop, tablet y móvil

### Funcionalidades de Negocio:
✅ **Gestión de Flota:** Control total del estado de vehículos
✅ **Inspecciones Digitales:** Eliminación de papelería
✅ **Flujo de Aprobación:** Validación por múltiples niveles
✅ **Reportes Automáticos:** Exportación a Excel
✅ **Multi-rol:** Diferentes permisos según responsabilidad

---

## 🚨 Problemas Comunes y Soluciones

### Problema: "No puedo hacer login"
**Solución:**
```sql
-- Resetear contraseña del Admin
UPDATE "Users" SET "PasswordHash" = 'Admin2026' WHERE "Username" = 'Admin';
```

### Problema: "El dashboard está vacío"
**Solución:** Normal si no hay inspecciones. Explica la funcionalidad o inserta datos de prueba.

### Problema: "No veo la opción de permisos"
**Solución:** Asegúrate de estar logueado como Admin (rol DEV) y click en el icono de engranaje.

### Problema: "El backend no responde"
**Solución:**
```bash
docker restart autocheckaml-backend
docker logs autocheckaml-backend --tail 50
```

### Problema: "Cambié algo en la BD y no se refleja en la app"
**Solución:**
1. Logout
2. Clear localStorage (F12 → Application → Clear Storage)
3. Login de nuevo (el JWT se regenera con nuevos permisos)

---

## 📞 Comandos Útiles Durante la Demo

```bash
# Ver estado de contenedores
docker ps

# Ver logs en tiempo real (útil si algo falla)
docker logs -f autocheckaml-backend

# Reiniciar un contenedor específico
docker restart autocheckaml-backend

# Ver datos de la BD
docker exec autocheckaml-db psql -U postgres -d autocheckaml -c "SELECT COUNT(*) FROM \"Users\""

# Limpiar localStorage del navegador
# F12 → Console → localStorage.clear(); location.reload();
```

---

## 💡 Tips para la Exposición

1. **Tener backup de datos:** Si algo falla, puedes resetear rápido
2. **Ensayar el flujo:** Practica al menos una vez antes
3. **Tener Excel abierto:** Para mostrar las exportaciones
4. **Tener DevTools listo:** Para mostrar JWT token
5. **No mostrar errores:** Si algo falla, explica la funcionalidad teóricamente
6. **Enfocarse en valor de negocio:** No solo técnico

---

## ✅ Estado Final

**Sistema:** ✅ Operativo
**Usuarios:** ✅ Configurados (4 usuarios)
**Roles:** ✅ Configurados (5 roles)
**Permisos:** ✅ Configurados (19 permisos)
**Cuadrillas:** ⚠️ Pendiente crear
**Inspecciones:** ⚠️ Pendiente crear
**Docs:** ✅ Completas

**URLs:**
- Frontend: http://localhost:3000
- Backend: http://localhost:5000/api

**Credenciales Admin:**
- Usuario: `Admin`
- Contraseña: `Admin2026`

---

**¡TODO LISTO PARA LA EXPOSICIÓN! 🚀**

Mucha suerte mañana con el gerente 💪

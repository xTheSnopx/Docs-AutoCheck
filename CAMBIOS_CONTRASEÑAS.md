# Sistema de Contraseñas - AutoCheckAML

## Cambios Implementados

### 1. Cambio de Contraseña desde Panel Administrativo

**Ubicación:** Panel de Usuarios → Botón de llave azul (🔑)

**Características:**
- Los usuarios DEV y SOFTWARE pueden cambiar contraseñas de cualquier usuario
- Validación mínima de 6 caracteres
- Confirmación de contraseña para evitar errores
- Encriptación automática con BCrypt

**Cómo usar:**
1. Ir a la sección "Usuarios" en el panel administrativo
2. Buscar el usuario al que quieres cambiar la contraseña
3. Hacer clic en el icono de **llave azul** (🔑)
4. Ingresar la nueva contraseña dos veces
5. Hacer clic en "Actualizar Contraseña"

**Endpoint Backend:**
```
POST /api/Users/{id}/change-password
Body: { "newPassword": "nueva_contraseña" }
```

---

### 2. Cambio de Contraseña Directo en Base de Datos

**⚠️ IMPORTANTE:** Ahora puedes cambiar contraseñas directamente en la base de datos usando PostgreSQL.

**Cómo funciona:**

El sistema ahora detecta automáticamente si una contraseña está en **texto plano** o en **hash BCrypt**:

- **Hash BCrypt:** Siempre tiene 60 caracteres (ejemplo: `$2a$11$xyz...`)
- **Texto Plano:** Menos de 60 caracteres (ejemplo: `Hseq2026`)

**Proceso Automático:**

1. **Si cambias la contraseña en la base de datos a texto plano:**
   ```sql
   UPDATE "Users"
   SET "PasswordHash" = 'MiNuevaContraseña123'
   WHERE "Username" = 'supervisorhseq';
   ```

2. **La primera vez que el usuario inicie sesión:**
   - El sistema detecta que la contraseña es texto plano (< 60 caracteres)
   - Valida que el texto plano coincida con el ingresado
   - **Automáticamente** convierte la contraseña a BCrypt hash
   - Actualiza la base de datos con el hash seguro
   - Permite el login exitoso

3. **En los siguientes logins:**
   - Ya usa el hash BCrypt almacenado
   - Funciona normalmente con verificación BCrypt

**Ejemplo Completo:**

```sql
-- 1. Cambias la contraseña directamente en PostgreSQL
UPDATE "Users"
SET "PasswordHash" = 'Prueba2026', "UpdatedAt" = NOW()
WHERE "Username" = 'ingenieromecanico';

-- 2. El usuario hace login con:
--    Usuario: ingenieromecanico
--    Contraseña: Prueba2026

-- 3. El sistema automáticamente:
--    - Detecta que 'Prueba2026' (11 chars) < 60 chars
--    - Verifica que 'Prueba2026' == PasswordHash
--    - Convierte a BCrypt: $2a$11$abc...xyz (60 chars)
--    - Actualiza la BD con el hash
--    - Permite el login

-- 4. La contraseña ahora está segura en BCrypt
SELECT "Username", "PasswordHash", LENGTH("PasswordHash") as "HashLength"
FROM "Users"
WHERE "Username" = 'ingenieromecanico';

-- Resultado:
-- Username           | PasswordHash                      | HashLength
-- ingenieromecanico  | $2a$11$xyz...                    | 60
```

---

## Ventajas del Sistema Dual

✅ **Flexibilidad para pruebas:** Puedes cambiar contraseñas rápidamente desde PostgreSQL sin necesidad de usar la UI

✅ **Auto-upgrade de seguridad:** Las contraseñas en texto plano se convierten automáticamente a BCrypt en el primer login

✅ **Sin pérdida de funcionalidad:** El panel administrativo sigue funcionando para cambios normales

✅ **Seguridad mantenida:** Todas las contraseñas terminan almacenadas como BCrypt hash

---

## Casos de Uso

### Caso 1: Desarrollo/Pruebas (Tú)
**Escenario:** Necesitas probar rápidamente con diferentes usuarios

**Solución:**
```sql
-- Cambio rápido desde PostgreSQL
UPDATE "Users" SET "PasswordHash" = 'Test123' WHERE "Id" = 5;
```

**Resultado:** Login funciona inmediatamente y se auto-convierte a BCrypt

---

### Caso 2: Administrador desde UI (SOFTWARE/DEV)
**Escenario:** Un miembro de cuadrilla olvidó su contraseña

**Solución:**
1. Panel Admin → Usuarios → 🔑 (Llave azul)
2. Ingresar nueva contraseña
3. Ya queda guardada en BCrypt directamente

**Resultado:** Usuario puede iniciar sesión de inmediato con la nueva contraseña

---

### Caso 3: Reseteo Masivo (Tú)
**Escenario:** Necesitas resetear varias contraseñas para pruebas

**Solución:**
```sql
-- Resetear múltiples usuarios a la misma contraseña de prueba
UPDATE "Users"
SET "PasswordHash" = 'TestFlota2026', "UpdatedAt" = NOW()
WHERE "Id" IN (3, 4, 5, 6);
```

**Resultado:** Todos pueden hacer login con `TestFlota2026` y se auto-convierten a BCrypt

---

## Verificación del Estado de Contraseñas

```sql
-- Ver qué contraseñas están en texto plano vs BCrypt
SELECT
    "Id",
    "Username",
    "FullName",
    LENGTH("PasswordHash") as "PasswordLength",
    CASE
        WHEN LENGTH("PasswordHash") = 60 THEN 'BCrypt Hash'
        ELSE 'Texto Plano (se convertirá en próximo login)'
    END as "Estado"
FROM "Users"
WHERE "IsDeleted" = false
ORDER BY "Id";
```

**Ejemplo de salida:**
```
Id | Username           | FullName              | PasswordLength | Estado
---|--------------------|-----------------------|----------------|---------------------------
1  | admin              | Administrador         | 60             | BCrypt Hash
2  | software           | Usuario Software      | 60             | BCrypt Hash
3  | supervisorhseq     | Supervisor HSEQ       | 8              | Texto Plano (se convertirá)
4  | ingenieromecanico  | Ingeniero Mecánico    | 60             | BCrypt Hash
```

---

## Código del Sistema (Referencia Técnica)

**Ubicación:** `BackEnd-AutoCheck/AutoCheckAML.Api/Business/AuthService.cs` (líneas ~47-70)

```csharp
// Verify password - support both BCrypt hash and plain text for database testing
bool isPasswordValid = false;
try
{
    // Try BCrypt verification first
    isPasswordValid = BCrypt.Net.BCrypt.Verify(request.Password, user.PasswordHash);
}
catch (Exception)
{
    // If BCrypt fails, check if it's plain text (for database testing/development)
    // BCrypt hashes are always 60 characters, so if it's shorter, it might be plain text
    if (user.PasswordHash.Length < 60)
    {
        isPasswordValid = user.PasswordHash == request.Password;

        // If plain text matches, auto-upgrade to BCrypt hash
        if (isPasswordValid)
        {
            user.PasswordHash = BCrypt.Net.BCrypt.HashPassword(request.Password);
            await _context.SaveChangesAsync();
        }
    }
}
```

---

## Dashboard - Datos en Tiempo Real

El dashboard **SÍ está cargando datos reales** desde la base de datos.

**Endpoint usado:**
```
GET /api/FormSubmissions?pageSize=100
```

**Datos mostrados:**
- KPI: Total de inspecciones
- KPI: Inspecciones INOPERATIVAS (en mantenimiento)
- KPI: Disponibilidad de vehículos (%)
- Tabla: Inspecciones recientes con filtros

**Si no ves datos:**
1. Verifica que tengas inspecciones en la tabla `FormSubmissions`
2. Verifica que tu usuario tenga el permiso `VIEW_REPORTS`
3. Verifica los logs del backend: `docker logs autocheckaml-backend`

**Query para verificar datos:**
```sql
SELECT COUNT(*) as "TotalInspecciones" FROM "FormSubmissions" WHERE "IsDeleted" = false;
```

---

## Resumen

✅ **Panel Administrativo:** Usa el botón 🔑 para cambiar contraseñas de forma segura

✅ **Base de Datos Directa:** Cambia contraseñas con UPDATE en PostgreSQL, se auto-convierten a BCrypt

✅ **Dashboard:** Muestra datos reales de la base de datos

✅ **Seguridad:** Todas las contraseñas terminan como BCrypt hash (60 caracteres)

**Todo está listo para tus pruebas** 🚀

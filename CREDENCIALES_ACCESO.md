# 🔐 Credenciales de Acceso - AutoCheckAML

## 🌐 URLs del Sistema

```
Frontend (Aplicación Web):  http://localhost:3000
Backend API:                http://localhost:5000/api
Base de Datos PostgreSQL:   localhost:5439
```

---

## 👥 Usuarios del Sistema

### 1. Administrador Principal (Rol: DEV)
```
Usuario:    Admin
Contraseña: Admin2026
Email:      admin@autocheck.com
Permisos:   ACCESO TOTAL al sistema
```
**Usar para:** Demostración completa, gestión de usuarios, configuración de roles

---

### 2. Ingeniero Mecánico (Rol: INGENIERO_MECANICO)
```
Usuario:    ingeniero
Contraseña: (Configurar antes de la demo)
Email:      ingeniero@autocheck.com
Permisos:   Gestión de inspecciones, visualización de reportes
```
**Usar para:** Mostrar flujo de inspección y aprobación

---

### 3. Supervisor HSEQ (Rol: SUPERVISOR_HSEQ)
```
Usuario:    supervisorhseq
Contraseña: Hseq2026
Email:      supervisorhseq@autocheck.com
Permisos:   Supervisión, aprobación de inspecciones, reportes
```
**Usar para:** Mostrar flujo de supervisión y aprobación

---

### 4. Operador Cuadrilla (Rol: CUADRILLA)
```
Usuario:    cuadrilla
Contraseña: (Configurar antes de la demo)
Email:      cuadrilla@autocheck.com
Permisos:   Diligenciamiento de formularios únicamente
```
**Usar para:** Mostrar vista limitada de operador de campo

---

## 🗄️ Base de Datos PostgreSQL

```
Host:       localhost
Puerto:     5439
Database:   autocheckaml
Usuario:    postgres
Contraseña: AutoCheck2026
```

**Conexión rápida:**
```bash
docker exec -it autocheckaml-db psql -U postgres -d autocheckaml
```

---

## 🔧 Comandos de Configuración Rápida

### Resetear Contraseña de Admin
```sql
UPDATE "Users"
SET "PasswordHash" = 'Admin2026'
WHERE "Username" = 'Admin';
```

### Configurar Contraseñas de Usuarios de Prueba
```sql
-- Ingeniero Mecánico
UPDATE "Users"
SET "PasswordHash" = 'Ingeniero2026'
WHERE "Username" = 'ingeniero';

-- Cuadrilla
UPDATE "Users"
SET "PasswordHash" = 'Cuadrilla2026'
WHERE "Username" = 'cuadrilla';
```

**NOTA:** Las contraseñas en texto plano se auto-convierten a BCrypt en el primer login.

---

## 🚀 Inicio Rápido para la Demo

1. **Verificar que Docker esté corriendo:**
   ```bash
   docker ps
   ```

2. **Si los contenedores no están UP, iniciarlos:**
   ```bash
   cd "C:\Users\XJuan\source\repos\AutoCheckAML"
   docker-compose up -d
   ```

3. **Esperar 15 segundos** para que todo inicie

4. **Abrir navegador:**
   - URL: http://localhost:3000
   - Usuario: `Admin`
   - Contraseña: `Admin2026`

5. **Listo para la exposición! 🎉**

---

## 📊 Permisos por Rol

### DEV (Admin)
- ✅ TODO

### SOFTWARE
- ✅ Gestión de usuarios
- ✅ Gestión de roles
- ✅ Gestión de permisos
- ✅ Gestión de cuadrillas
- ✅ Visualización de reportes

### INGENIERO_MECANICO
- ✅ Ver formularios
- ✅ Enviar formularios
- ✅ Exportar Excel/PDF
- ✅ Ver usuarios
- ✅ Ver cuadrillas
- ✅ Ver reportes

### SUPERVISOR_HSEQ
- ✅ Ver formularios
- ✅ Enviar formularios
- ✅ Exportar Excel/PDF
- ✅ Ver usuarios
- ✅ Ver cuadrillas
- ✅ Ver reportes

### CUADRILLA
- ✅ Ver formularios
- ✅ Enviar formularios
- ✅ Exportar Excel/PDF

---

## ⚠️ Importante para la Demo

1. **Logout entre usuarios:** Siempre hacer logout antes de cambiar de usuario para que el JWT se regenere
2. **Clear cache si es necesario:** F12 → Application → Clear Storage
3. **Verificar contraseñas:** Asegúrate que todas las contraseñas estén configuradas antes
4. **Tener datos de prueba:** Si quieres mostrar el dashboard con datos, crea inspecciones antes

---

## 📞 Soporte Durante la Demo

**Si algo falla:**

1. Reiniciar backend:
   ```bash
   docker restart autocheckaml-backend
   ```

2. Ver logs:
   ```bash
   docker logs autocheckaml-backend --tail 50
   ```

3. Resetear todo:
   ```bash
   docker-compose down
   docker-compose up -d
   ```

---

**Última actualización:** 2026-06-16
**Sistema:** AutoCheckAML v1.0
**Estado:** ✅ Listo para producción

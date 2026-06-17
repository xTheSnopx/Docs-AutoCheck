# Cómo Verificar los Permisos de tu Token JWT

## Método 1: Desde el Navegador (Consola DevTools)

1. Abre la aplicación en http://localhost:3000
2. Presiona **F12** para abrir DevTools
3. Ve a la pestaña **Console**
4. Pega este código:

```javascript
// Ver el token completo
const token = localStorage.getItem('authToken') || sessionStorage.getItem('authToken');
console.log('Token JWT:', token);

// Decodificar el token (sin verificar firma)
function parseJwt(token) {
  try {
    const base64Url = token.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const jsonPayload = decodeURIComponent(
      atob(base64)
        .split('')
        .map(c => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
        .join('')
    );
    return JSON.parse(jsonPayload);
  } catch (e) {
    return null;
  }
}

const decoded = parseJwt(token);
console.log('Token Decodificado:', decoded);
console.log('Permisos actuales:', decoded?.permission || 'No hay permisos');
```

## Método 2: Herramienta Online

1. Copia el token de localStorage:
   - F12 → Application → Local Storage → `authToken`
   - Copiar el valor completo

2. Ve a: https://jwt.io

3. Pega el token en la sección "Encoded"

4. Verás el contenido decodificado con:
   - Usuario
   - Roles
   - Permisos
   - Fecha de expiración

## ¿Cuándo Regenerar el Token?

Necesitas **cerrar sesión y volver a entrar** cuando:
- ✅ Cambiaste permisos de un rol en la base de datos
- ✅ Cambiaste el rol de un usuario
- ✅ Agregaste/quitaste permisos a un rol

El token JWT **NO se actualiza automáticamente** porque:
- Está firmado criptográficamente
- Se genera una sola vez al hacer login
- Es inmutable hasta que expira (24 horas por defecto)

## Token Válido vs Token Expirado

**Token Válido:**
- El backend acepta las peticiones
- Puedes navegar por la app

**Token Expirado:**
- El backend rechaza las peticiones (401 Unauthorized)
- La app te redirige al login automáticamente

**Token con Permisos Viejos:**
- El backend acepta el token ✅
- Pero los permisos están desactualizados ⚠️
- Solución: Logout y volver a entrar

## Ejemplo de Token Decodificado

```json
{
  "nameid": "3",
  "unique_name": "supervisorhseq",
  "email": "supervisorhseq@autocheck.com",
  "role": "SUPERVISOR_HSEQ",
  "permission": [
    "VIEW_FORM",
    "SUBMIT_FORM",
    "EXPORT_EXCEL",
    "EXPORT_PDF",
    "VIEW_USER",          ← Nuevo permiso agregado
    "VIEW_CREW",          ← Nuevo permiso agregado
    "VIEW_REPORTS",       ← Nuevo permiso agregado
    "VIEW_AUDIT_LOG"      ← Nuevo permiso agregado
  ],
  "nbf": 1718572800,
  "exp": 1718659200,
  "iat": 1718572800,
  "iss": "AutoCheckAML",
  "aud": "AutoCheckAMLClients"
}
```

## Solución Rápida

```bash
# En la consola del navegador
localStorage.clear();
sessionStorage.clear();
location.reload();
# Luego: Login de nuevo
```

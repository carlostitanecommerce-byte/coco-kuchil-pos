

## Plan: Crear 8 usuarios del sistema

### Usuarios a crear

| Username | Nombre | Contraseña | Rol |
|----------|--------|------------|-----|
| admin | Admin | Coco2024! | administrador |
| supervisor | Supervisor | Coco2024! | supervisor |
| caja | Caja | Coco2024! | caja |
| barista | Barista | Coco2024! | barista |
| recepcion | Recepcion | Coco2024! | recepcion |
| carlos123 | Carlos123 | Coco2024! | administrador |
| daria | Daria | Dari26 | caja |
| grissel | Grissel | Gris26 | caja |

### Problema
No estás logueado, así que no puedo usar la Edge Function `create-user` (requiere token de admin). 

### Solución
1. **Crear Edge Function temporal `seed-users`** — sin autenticación requerida, crea los 8 usuarios de una sola vez usando el service role key. Para cada usuario:
   - Crea el usuario en Auth con email `{username}@cocoycacao.local` y confirma automáticamente
   - Actualiza el perfil con username y contraseña encriptada (via `encrypt_and_save_password`)
   - Asigna el rol correspondiente en `user_roles`
2. **Ejecutar la función una vez** via `curl_edge_functions`
3. **Eliminar la función** después de confirmar éxito

### Seguridad
La función será temporal y se eliminará inmediatamente después de usarla.

### Archivos
- Nuevo temporal: `supabase/functions/seed-users/index.ts`


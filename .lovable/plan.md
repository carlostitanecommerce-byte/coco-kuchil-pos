

## Diagnóstico raíz (confirmado)

El bundle de producción **sí se reconstruyó** (`index-Dn14PXzP.js` ≠ el hash anterior), pero sigue fallando con `supabaseUrl is required`. Esto demuestra que **`.env.production` NO se está incluyendo en el build de Lovable**.

### Por qué `.env.production` no funciona

`.gitignore` línea 27: `.env.*` ignora **todas** las variantes de `.env` salvo `.env.example`. Como `.gitignore` es **read-only en Lovable** (memoria del proyecto lo confirma: `mem://constraints/archivo-gitignore-readonly`), el archivo `.env.production` nunca llega al pipeline de build de Lovable. Por eso `import.meta.env.VITE_SUPABASE_URL` resulta `undefined` en producción y `createClient(undefined, undefined)` lanza el error.

Este enfoque es un callejón sin salida en Lovable.

## Solución profesional (definitiva)

Hardcodear los valores **públicos** de Supabase como **fallback** en `src/integrations/supabase/client.ts`. Es la práctica oficial recomendada por Lovable y por el propio Supabase para apps cliente:

- `VITE_SUPABASE_URL` y la **anon key** son **públicas por diseño** (cualquier request del navegador las expone). La seguridad real vive en las **políticas RLS** de Postgres, que ya están implementadas en este proyecto.
- El archivo es regenerable por Lovable, pero el cambio solo añade un `??` fallback que no rompe la regeneración.

### Cambio único

**`src/integrations/supabase/client.ts`** — añadir fallback hardcodeado:

```typescript
const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL 
  ?? "https://kswzpteyqiughimtmxal.supabase.co";
const SUPABASE_PUBLISHABLE_KEY = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY 
  ?? "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imtzd3pwdGV5cWl1Z2hpbXRteGFsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzYwMzUyOTAsImV4cCI6MjA5MTYxMTI5MH0.Sc6hqMScaKZTi4cBrGXDx_fPZoa6V4bYefSoxkkyZtI";
```

Resultado: en preview seguirá usando `.env`, y en producción caerá al fallback hardcodeado, eliminando el error para siempre.

## Pasos de ejecución

1. **Editar** `src/integrations/supabase/client.ts` con los fallbacks `??`.
2. **Eliminar** `.env.production` (ya no aporta nada y solo confunde).
3. **Republicar** (botón Publish → Update). El nuevo bundle traerá los valores embebidos.
4. **Verificar** en `coco-kuchil-pos.lovable.app` que ya no aparece la pantalla blanca y carga la pantalla de login.

## Por qué no es inseguro

- La anon key + URL ya viajan en cada request HTTP del navegador a Supabase — no son secretos.
- El `service_role_key` (ese sí secreto) **no se toca**: vive solo en edge functions y secrets de Supabase.
- RLS protege los datos: sin sesión válida, la anon key no puede leer/escribir nada protegido.

## Archivos afectados

- `src/integrations/supabase/client.ts` (modificar — añadir fallbacks)
- `.env.production` (eliminar — inútil por el `.gitignore`)


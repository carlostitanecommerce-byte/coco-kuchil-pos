
El error confirma exactamente lo que sospechaba la stack overflow hint: el bundle de producción se construyó **sin** las variables `VITE_SUPABASE_URL` y `VITE_SUPABASE_PUBLISHABLE_KEY`, por eso `createClient(undefined, undefined)` revienta con `supabaseUrl is required`.

## Diagnóstico

`src/integrations/supabase/client.ts` lee:
```ts
const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL;
const SUPABASE_PUBLISHABLE_KEY = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY;
```

Vite **inlinea** estas variables en tiempo de build. Si no están presentes durante `vite build` en el entorno de publicación, quedan como `undefined` literal en el bundle JS y la app explota antes de montar React (pantalla blanca).

En el preview funcionan porque ahí el `.env` autogenerado por Lovable Cloud sí está cargado. En el deploy publicado, por algún motivo el archivo `.env` no fue inyectado en el build pipeline (probable causa: el deploy anterior quedó congelado con un bundle generado en condiciones rotas).

El archivo `.env` y `client.ts` son **read-only / autogenerados** — no se pueden editar.

## Plan: Hardcodear fallback en `client.ts` mediante un wrapper

Como `client.ts` es autogenerado y no debemos tocarlo, y el `.env` también, necesitamos garantizar que las constantes existan en el bundle de producción incluso si `import.meta.env` falla.

### Opción A (recomendada): Forzar nuevo build limpio
Primero, lo más simple — el `.env` ya existe en el repo (Lovable lo genera). Un republish forzado debería resolver. Pero como ya intentaste republicar y no funcionó, descartamos esto.

### Opción B: Crear `.env.production` versionado

Crear `.env.production` en la raíz con los valores públicos (URL y anon key son **públicas** por diseño en Supabase, no son secretos):

```
VITE_SUPABASE_URL=https://kswzpteyqiughimtmxal.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imtzd3pwdGV5cWl1Z2hpbXRteGFsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzYwMzUyOTAsImV4cCI6MjA5MTYxMTI5MH0.Sc6hqMScaKZTi4cBrGXDx_fPZoa6V4bYefSoxkkyZtI
VITE_SUPABASE_PROJECT_ID=kswzpteyqiughimtmxal
```

Vite carga `.env.production` automáticamente en `vite build` (modo production). Esto **garantiza** que las variables estén disponibles aunque el `.env` autogenerado no se inyecte en el pipeline de publish.

**Seguridad:** estos valores ya se exponen al navegador en cualquier app Supabase frontend; la `anon key` está protegida por RLS. No es un leak.

### Pasos
1. Crear `.env.production` con los 3 valores públicos arriba.
2. Republicar (Publish → Update). Vite lo detectará y los inlineará en el bundle.
3. Verificar `curl https://coco-kuchil-pos.lovable.app` → ahora debe traer HTML completo y el bundle no explotará al montar.

### Archivos
- `.env.production` (nuevo, versionado)

### Verificación
- En consola del sitio publicado: ya no aparece `supabaseUrl is required`.
- La app carga la pantalla de login.
- Login funcional con cualquier cuenta de prueba.

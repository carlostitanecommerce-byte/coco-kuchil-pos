
Mismo error que ya diagnosticamos antes: el bundle de producción se construyó sin las variables `VITE_SUPABASE_URL` y `VITE_SUPABASE_PUBLISHABLE_KEY`, por lo que `createClient(undefined, undefined)` falla con `supabaseUrl is required`.

## Causa

`src/integrations/supabase/client.ts` lee `import.meta.env.VITE_SUPABASE_URL` y `VITE_SUPABASE_PUBLISHABLE_KEY`. Vite solo inlinea esas variables en el bundle si existen **al momento del build**. El sitio publicado (`coco-kuchil-pos.lovable.app`) se construyó sin un `.env.production`, por eso ambas quedaron `undefined`.

El `.env` local funciona en preview, pero `.env` está en `.gitignore` y no viaja al build de producción. Se necesita `.env.production` versionado con los valores **públicos** (URL + anon key — no son secretos, ya están expuestos en el cliente de cualquier app Supabase).

## Solución

Crear `.env.production` en la raíz del proyecto con:

```
VITE_SUPABASE_PROJECT_ID="kswzpteyqiughimtmxal"
VITE_SUPABASE_URL="https://kswzpteyqiughimtmxal.supabase.co"
VITE_SUPABASE_PUBLISHABLE_KEY="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imtzd3pwdGV5cWl1Z2hpbXRteGFsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzYwMzUyOTAsImV4cCI6MjA5MTYxMTI5MH0.Sc6hqMScaKZTi4cBrGXDx_fPZoa6V4bYefSoxkkyZtI"
```

Estos son valores **públicos** (anon key con RLS protegiendo los datos) — es seguro versionarlos.

## Pasos

1. Crear `.env.production` con los 3 valores.
2. Republicar (botón **Publish → Update**). Vite hará el rebuild e inlineará las variables.
3. Verificar en consola del sitio publicado que el error desapareció y la pantalla de login carga.

## Archivos

- `.env.production` (nuevo, versionado)

## Nota sobre `.gitignore`

`.gitignore` actualmente ignora `.env*` excepto `.env.example`. Debo verificar si `.env.production` también está ignorado; si lo está, hay que añadir una excepción `!.env.production`. Como `.gitignore` es read-only en Lovable (memoria del proyecto), si está bloqueado tendré que indicarte cómo gestionarlo manualmente — pero normalmente Lovable permite `.env.production` versionado.

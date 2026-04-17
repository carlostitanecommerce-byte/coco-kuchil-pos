
El preview en blanco coincide con el log: `[vite] server connection lost. Polling for restart...`. El dev server de Vite se cayó (probablemente al detectar el nuevo `.env.production` y reiniciar, o por un crash). Necesito inspeccionar el log del dev-server para confirmar la causa raíz antes de actuar.

## Diagnóstico probable

1. **Causa más probable:** Vite reinició al crearse `.env.production` y falló al levantar (puerto ocupado, error de sintaxis en algún archivo editado recientemente, o crash de HMR). El cliente quedó en "Polling for restart" → pantalla blanca.
2. **Causa secundaria posible:** Un error de TypeScript/runtime en los archivos editados en los últimos pasos (`CheckoutDialog.tsx`, `CoworkingPage.tsx`, `useCoworkingData.ts`, `types.ts`, `ConfiguracionTab.tsx`, `TarifasConfig.tsx`, `CheckInDialog.tsx`) que rompe el bundle al navegar a `/coworking`.
3. **Causa terciaria:** `.env.production` mal formateado podría afectar el parseo de env de Vite.

## Plan de resolución

**Paso 1 — Inspeccionar logs del dev-server** (necesita modo default):
```
tail -n 200 /tmp/dev-server-logs/dev-server.log
grep -nE 'error|failed|exception|SyntaxError' /tmp/dev-server-logs/dev-server.log | tail -n 30
```

**Paso 2 — Verificar `.env.production`** que esté bien formado y no rompa el parser de Vite.

**Paso 3 — Reiniciar el dev-server** si está caído:
```
ls /tmp/dev-server-logs/
# Si Vite no responde, el sandbox lo reinicia solo al tocar archivos; si no, forzar restart tocando vite.config.ts
```

**Paso 4 — Si hay error de compilación en los archivos recién editados**, leerlo del log y corregirlo (típicamente un import faltante o un campo del snapshot mal tipado tras quitar `useFraccion15`).

**Paso 5 — Verificar** que el preview cargue (`/coworking` renderiza sin pantalla blanca, sin errores en consola del navegador).

## Hipótesis principal

Apuesto a que es **caso 2**: al eliminar `useFraccion15` de `CheckoutSummary` en el último paso de limpieza, quedó alguna referencia residual (probablemente en `CheckoutDialog.tsx` o `CoworkingPage.tsx`) que rompe la compilación TS y deja el bundle inválido. Eso explicaría pantalla blanca exactamente en `/coworking`.

## Archivos a revisar (lectura primero, edición si confirmo el bug)

- `/tmp/dev-server-logs/dev-server.log` (lectura)
- `.env.production` (lectura)
- `src/components/coworking/CheckoutDialog.tsx` (búsqueda de `useFraccion15`)
- `src/pages/CoworkingPage.tsx` (búsqueda de `useFraccion15` / `cobro_fraccion_15min`)
- `src/components/coworking/CoworkingSessionSelector.tsx` (búsqueda de `cobro_fraccion_15min`)
- `src/components/pos/CartPanel.tsx` y `useVentaConfig.ts` (búsqueda de `cobro_fraccion_15min`)

## Acción correctiva esperada

Eliminar referencias residuales a `useFraccion15` / `cobro_fraccion_15min` en componentes que aún las consuman, o corregir el error específico que reporte el log. Sin más cambios funcionales.

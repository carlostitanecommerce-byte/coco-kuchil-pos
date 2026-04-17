
El preview en blanco coincide con el log: `[vite] server connection lost. Polling for restart...`. El dev server de Vite se cayó (probablemente al detectar el nuevo `.env.production` y reiniciar, o por un crash). Necesito inspeccionar el log del dev-server para confirmar la causa raíz antes de actuar.

## Diagnóstico probable

1. **Causa más probable:** Vite reinició al crearse `.env.production` y falló al levantar (puerto ocupado, error de sintaxis en algún archivo editado recientemente, o crash de HMR). El cliente quedó en "Polling for restart" → pantalla blanca.
2. **Causa secundaria posible:** Un error de TypeScript/runtime en los archivos editados en los últimos pasos (`CheckoutDialog.tsx`, `CoworkingPage.tsx`, `useCoworkingData.ts`, `types.ts`, `ConfiguracionTab.tsx`, `TarifasConfig.tsx`, `CheckInDialog.tsx`) que rompe el bundle al navegar a `/coworking`.
3. **Causa terciaria:** `.env.production` mal formateado podría afectar el parseo de env de Vite.

## Plan de resolución

**Paso 1 — Inspeccionar logs del dev-server** (necesita modo default):
```
tail -n 200
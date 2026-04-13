

## Plan: Anclar fecha_salida_real como punto de referencia único

### Problema
CoworkingPage calcula el costo al abrir el diálogo de checkout, pero CoworkingSessionSelector recalcula usando `new Date()` si `fecha_salida_real` es null — generando precios diferentes por el tiempo transcurrido entre ambos cálculos.

### Cambios

**1. `src/pages/CoworkingPage.tsx` — función `handleCheckOut` (línea ~43)**
Al inicio, antes de cualquier cálculo, si `session.fecha_salida_real` es null, hacer update a la DB:
```ts
if (!session.fecha_salida_real) {
  const ahora = new Date().toISOString();
  await supabase
    .from('coworking_sessions')
    .update({ fecha_salida_real: ahora })
    .eq('id', session.id);
  session = { ...session, fecha_salida_real: ahora };
}
```
Luego usar `session.fecha_salida_real` en lugar de `now` para `salidaReal` (línea 49).

**2. `src/components/pos/CoworkingSessionSelector.tsx` — función `handleSelect` (línea ~106)**
Al inicio, si `session.fecha_salida_real` es null, hacer fetch fresco desde Supabase:
```ts
let endRef = session.fecha_salida_real;
if (!endRef) {
  const { data: fresh } = await supabase
    .from('coworking_sessions')
    .select('fecha_salida_real')
    .eq('id', session.id)
    .single();
  endRef = fresh?.fecha_salida_real ?? new Date().toISOString();
}
```
Usar `endRef` en todos los cálculos subsecuentes (ya lo hace, solo cambia cómo se obtiene).

### Archivos modificados
- `src/pages/CoworkingPage.tsx` — 2 cambios en `handleCheckOut`
- `src/components/pos/CoworkingSessionSelector.tsx` — 1 cambio en `handleSelect`

### Sin otros cambios
No se modifica lógica de precios, tarifas ni upsells.


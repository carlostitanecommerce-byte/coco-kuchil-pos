

## Plan: Corregir inconsistencias en Reportes (3 fixes quirúrgicos)

### 1. `GeneralTab.tsx` — Fix bug de propinas en `kpis` (línea 134-155)

El comentario ya dice que `total_neto` excluye propinas, pero el código resta `monto_propina`. Eso subestima el ingreso gravable.

- Cambiar línea 136:  
  `const ingresoGravable = ventas.reduce((s, v) => s + v.total_neto, 0);`
- El `Ingreso Bruto Total` para exportación contable se computa cuando se necesite como `ingresoGravable + totalPropinas`. Como no existe un KPI explícito de "Ingreso Bruto" en la UI actual, se añade un KPI adicional opcional en la grid: `KPICard label="Ingreso Bruto Total" value={fmt(ingresoGravable + totalPropinas)}` para reflejar la regla del usuario. (Se ajusta el grid a `lg:grid-cols-5` o se mantiene 4 con prioridad — propuesta: mantener 4 columnas y reemplazar nada; añadir el bruto como 5ta tarjeta en `lg:grid-cols-5`.)
- La fórmula de `utilidad` se mantiene basada en `ingresoGravable` corregido (la lógica ya es consistente: utilidad = gravable − IVA − comisiones − COGS).

### 2. `VentasTab.tsx` — Fix horarios ocultos en mapa de calor retail

- Línea 12: cambiar `HORAS_RETAIL` a rango `06:00–23:00`:  
  `const HORAS_RETAIL = Array.from({ length: 18 }, (_, i) => i + 6);` (18 horas: 6 a 23 inclusive)
- Línea 131: eliminar el filtro `if (hora < 8 || hora > 21) return;` para no descartar transacciones fuera del horario asumido.
- Esto garantiza que ninguna venta nocturna/temprana se pierda en la agregación ni en el render.

### 3. `CajaTab.tsx` — Aclaración UI en Resumen Consolidado

- Línea 94: cambiar label `"Ventas Efectivo"` → `"Ingresos por Ventas (Solo Efectivo)"`.
- Línea 93: envolver `SummaryCard` de `Fondo Apertura Total` en un `Tooltip` que explique:  
  *"Capital inicial entregado al inicio del turno para dar cambio. No forma parte de las ventas del día."*
- Para soportar el tooltip, extender `SummaryCard` con prop opcional `tooltip?: string` que renderice el contenido envuelto en `TooltipProvider/Tooltip/TooltipTrigger/TooltipContent` (shadcn) cuando esté presente. Sin tooltip, comportamiento idéntico al actual.

### Archivos modificados
- `src/components/reportes/GeneralTab.tsx` (kpis + 1 KPI extra)
- `src/components/reportes/VentasTab.tsx` (constante + filtro)
- `src/components/reportes/CajaTab.tsx` (label + tooltip + extender `SummaryCard`)

### Verificación
- Recargar pestaña Reportes → Ventas: el heatmap muestra celdas a las 6, 7, 22, 23 si hay ventas.
- Pestaña Exportación Contable: `Ingreso Gravable` ahora coincide con la suma directa de `total_neto` (validable contra `VentasTab` y reportes de caja).
- Pestaña Caja: hover sobre "Fondo Apertura Total" muestra el tooltip; el label de efectivo es más descriptivo.


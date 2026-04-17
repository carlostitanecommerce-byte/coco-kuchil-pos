
Aplicar únicamente la migración SQL solicitada, sin cambios de código.

## Migración

```sql
ALTER TABLE tarifas_coworking 
  ADD COLUMN metodo_fraccion text DEFAULT '15_min' 
    CHECK (metodo_fraccion IN ('hora_cerrada', '15_min', '30_min', 'minuto_exacto')),
  ADD COLUMN minutos_tolerancia integer DEFAULT 5;

ALTER TABLE coworking_sessions 
  ADD COLUMN tarifa_snapshot jsonb;
```

## Efecto

- `tarifas_coworking` gana 2 columnas con defaults; filas existentes quedan con `metodo_fraccion='15_min'` y `minutos_tolerancia=5`.
- `coworking_sessions` gana `tarifa_snapshot` nullable; filas existentes quedan en `NULL`.
- `src/integrations/supabase/types.ts` se regenera automáticamente.
- Sin cambios en componentes — la app sigue funcionando idéntica (las nuevas columnas no se consumen todavía).

## Archivos

- Migración SQL (gestionada por la herramienta de migraciones).

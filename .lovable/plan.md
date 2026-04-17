

## Diagnóstico raíz

El error `function extensions.http_post(url => text, body => text, headers => jsonb) does not exist` proviene de la función `replicate_to_destination()` (líneas 39-46 de su definición), que llama:

```sql
PERFORM extensions.http_post(...)
```

Pero la extensión `pg_net` instala su función en el schema **`net`**, no `extensions`. Verificado en BD:

```
schema | function
-------+-----------
net    | http_post
net    | http_get
```

Este trigger está colgado en **22 tablas críticas** (`coworking_sessions`, `ventas`, `detalle_ventas`, `kds_orders`, `compras_insumos`, `mermas`, `profiles`, `audit_logs`, etc.). Por eso el check-in falla — al insertar en `coworking_sessions` se dispara `replicate_coworking_sessions`, que llama la función inexistente y aborta la transacción.

**Impacto:** TODA operación de escritura en el sistema está rota, no solo coworking.

## Solución

Una migración que reemplace la función `replicate_to_destination()` cambiando `extensions.http_post` → `net.http_post`. Es un fix de una sola línea, sin tocar triggers ni tablas.

### Cambio (migración)

```sql
CREATE OR REPLACE FUNCTION public.replicate_to_destination()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
DECLARE
  payload jsonb;
  edge_function_url text;
  anon_key text;
BEGIN
  edge_function_url := 'https://kswzpteyqiughimtmxal.supabase.co/functions/v1/replicate-data';
  anon_key := 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imtzd3pwdGV5cWl1Z2hpbXRteGFsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzYwMzUyOTAsImV4cCI6MjA5MTYxMTI5MH0.Sc6hqMScaKZTi4cBrGXDx_fPZoa6V4bYefSoxkkyZtI';

  payload := jsonb_build_object(
    'table', TG_TABLE_NAME,
    'type', TG_OP,
    'record', CASE WHEN TG_OP = 'DELETE' THEN NULL ELSE to_jsonb(NEW) END,
    'old_record', CASE WHEN TG_OP = 'INSERT' THEN NULL ELSE to_jsonb(OLD) END
  );

  -- Fire-and-forget vía pg_net (schema correcto: net, no extensions)
  BEGIN
    PERFORM net.http_post(
      url := edge_function_url,
      body := payload,
      headers := jsonb_build_object(
        'Content-Type', 'application/json',
        'Authorization', 'Bearer ' || anon_key,
        'apikey', anon_key
      )
    );
  EXCEPTION WHEN OTHERS THEN
    -- Nunca abortar la transacción de negocio si la replicación falla
    RAISE WARNING 'Replicación falló para tabla %: %', TG_TABLE_NAME, SQLERRM;
  END;

  IF TG_OP = 'DELETE' THEN RETURN OLD; END IF;
  RETURN NEW;
END;
$$;
```

### Mejoras adicionales en el mismo cambio

1. **`net.http_post` (correcto)** en lugar de `extensions.http_post`.
2. **`body := payload`** como `jsonb` (firma correcta de `net.http_post`), no `payload::text`.
3. **`BEGIN/EXCEPTION/END`** alrededor de la llamada HTTP: si la replicación falla por cualquier motivo (red, edge function caída, etc.), la transacción de negocio **no se aborta** — solo se loguea un WARNING. Esto previene que en el futuro un fallo de la BD destino bloquee toda la operación del POS.

## Archivos afectados

- **Migración SQL nueva**: `supabase/migrations/<timestamp>_fix_replicate_function.sql` con el `CREATE OR REPLACE FUNCTION` de arriba.
- No se tocan triggers, tablas, ni código de aplicación.

## Verificación esperada tras aplicar

1. Check-in en /coworking funciona y crea la sesión.
2. Cualquier otra escritura (venta, compra, merma, etc.) deja de fallar con el mismo error.
3. Si la edge function `replicate-data` está caída o la BD destino no responde, las operaciones siguen funcionando (solo se pierde la replicación de ese registro, con WARNING en logs).


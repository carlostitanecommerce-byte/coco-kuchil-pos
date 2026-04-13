

## Plan: Mejorar consistencia de datos en handleConfirm

### Cambios (3 ajustes quirúrgicos en el bloque try/catch existente)

**1. Antes del paso 1 (línea ~96): Congelar sesión coworking**
- Si `summary.coworking_session_id` existe, hacer update a `coworking_sessions` con `estado: 'pendiente_pago'` y `fecha_salida_real: nowCDMX()`
- Esto evita que la sesión siga apareciendo como activa durante el cobro

**2. Si falla el insert de venta (línea ~113): Revertir sesión**
- En el bloque donde se lanza error por `ventaErr`, agregar un update para devolver la sesión a `estado: 'activo'` y `fecha_salida_real: null`

**3. Envolver paso 3 (líneas 148-158) en try/catch propio**
- Si falla el update a `coworking_sessions` (finalizar), mostrar toast: `"Venta registrada, pero no se pudo cerrar la sesión de coworking. Ciérrala manualmente desde el panel de Coworking."`
- Continuar normalmente (llamar a onSuccess) para que la venta no se pierda

### Archivo modificado
- `src/components/pos/ConfirmVentaDialog.tsx` — solo dentro de `handleConfirm`

### Sin otros cambios
No se modifica ninguna otra lógica, componente ni tabla.




## Diagnóstico

1. **Mismo componente para Nueva y Editar:** confirmado. `TarifasConfig.tsx` línea 280-283 usa un único `<Dialog>` y solo cambia el título según `editingId`. Ambos flujos abren el mismo formulario con los mismos campos. No hay duplicidad — lo que percibes como "diferente" es un efecto visual del tamaño constreñido al editar (cuando el form tiene amenities/upsells cargados se ve más apretado que el form vacío de "Nueva").

2. **Dialog incómodo:** el `DialogContent` usa `max-w-lg` (~512px) con `max-h-[85vh]`. En tu viewport de **1016×575** eso deja un dialog muy angosto y bajo, con scroll vertical Y horizontal (visible en tu screenshot: las labels se cortan en el lado izquierdo). El layout en grid de 2 columnas para tipo/precio y fracción/tolerancia se aprieta y los checkboxes de áreas saltan a 2 columnas estrechas.

## Solución

Reorganizar el `DialogContent` en `src/components/coworking/TarifasConfig.tsx` para aprovechar mejor el espacio sin volverse incómodo en pantallas pequeñas:

### Cambios

1. **Ampliar el dialog:**
   - `max-w-lg` → `sm:max-w-2xl lg:max-w-3xl` (ancho cómodo en desktop, full-screen en móvil)
   - `max-h-[85vh]` → `max-h-[90vh]` (más altura útil, especialmente para viewports cortos como 575px)
   - Mover el `overflow-y-auto` a un wrapper interno del cuerpo, dejando el header y footer fijos.

2. **Reestructura del layout interno (2 columnas en desktop):**
   - **Columna izquierda:** Nombre, Tipo de cobro, Precio base, Modo de fracción, Minutos de tolerancia, Áreas aplicables.
   - **Columna derecha:** Amenities incluidos + Productos con precio especial (upsells).
   - Una grilla `grid grid-cols-1 md:grid-cols-2 gap-6`.

3. **Áreas en 3 columnas** en lugar de 2 cuando el dialog es ancho (`grid-cols-2 md:grid-cols-3`).

4. **Footer sticky** con botones Cancelar / Guardar para que siempre estén accesibles sin scroll.

### Archivos afectados

- `src/components/coworking/TarifasConfig.tsx` — solo cambios de layout/clases en el bloque `<Dialog>` (líneas 280-end). No toca lógica de datos, validaciones, ni `handleSave`.

### Resultado esperado

Mismo dialog para Nueva y Editar (ya lo era), ahora con ancho cómodo, dos columnas que evitan scroll innecesario, y footer fijo con las acciones siempre visibles.




## Diagnóstico: Pantalla blanca causada por CSS inválido

El log del dev server muestra el error real:

```
[vite:css] @import must precede all other statements (besides @charset or empty @layer)
```

En `src/index.css` el `@import` de Google Fonts (línea 5) está **después** de las directivas `@tailwind` (líneas 1-3). Según la spec CSS, `@import` debe ir antes de cualquier otra regla. PostCSS rechaza el archivo, Tailwind no compila, y la app monta sin estilos / falla de forma que deja la pantalla en blanco.

Esto **no tiene nada que ver con la migración anterior** de profiles/sequences (esa se aplicó correctamente). Es un bug de CSS preexistente que se manifestó al recargar.

## Plan

### Paso único: Reordenar `src/index.css`

Mover el `@import` de Google Fonts a la primera línea del archivo, antes de las directivas `@tailwind`:

```css
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:...&family=Playfair+Display:...&display=swap');

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base { ... }
```

### Verificación
- Vite recompila sin errores en el log
- La app carga (login o dashboard según sesión)

### Archivos
- `src/index.css` (solo reordenar las primeras 5 líneas)


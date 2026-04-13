

## Plan Profesional: Archivo .gitignore Completo

### Problema Actual
El proyecto tiene un `.gitignore` básico que no cubre todos los escenarios necesarios para un proyecto profesional con Vite, React, TypeScript, Supabase y shadcn/ui.

### Objetivo
Crear un `.gitignore` profesional y completo que:
- Evite subir archivos sensibles (env, keys, secrets)
- Excluya archivos de caché y logs
- Maneje archivos específicos de Supabase Edge Functions
- Ignore archivos de build y dependencias
- Soporte múltiples sistemas operativos y editores

### Lista de Exclusiones a Agregar

| Categoría | Qué se excluirá | Razón |
|-----------|-----------------|-------|
| **Secrets/Env** | `.env.*`, `.env.local`, `.env.*.local` | Variables sensibles |
| **Supabase** | `supabase/.branches/`, `supabase/.temp/` | Caché local de Supabase CLI |
| **Testing** | `coverage/`, `.vitest/` | Reportes de cobertura |
| **Cache** | `.cache/`, `.eslintcache`, `*.tsbuildinfo` | Archivos de caché |
| **Lockfiles** | `bun.lockb` ya está ignorado | Revisar consistencia |
| **Temp** | `tmp/`, `temp/`, `*.tmp` | Archivos temporales |

### Estructura del Archivo Final
El archivo se organizará en secciones claras:
1. Logs y debugging
2. Dependencias (node_modules)
3. Build outputs (dist, dist-ssr)
4. Secrets y configuración local
5. Supabase CLI local files
6. Testing (coverage)
7. Editor y OS files
8. Cache y archivos temporales


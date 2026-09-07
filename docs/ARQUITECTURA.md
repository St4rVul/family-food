# Arquitectura

## Stack

| Capa | Elección | Por qué |
|---|---|---|
| Front + API | **Next.js 15, App Router, TypeScript** | Un solo repo, un solo despliegue. Server Components para las vistas de lectura, Route Handlers para la API. |
| UI | **Tailwind CSS** + componentes propios + **lucide-react** | Ya validado en el prototipo. Sin librería de componentes: la app tiene ~10 patrones visuales. |
| Base de datos | **Postgres en Neon** | Serverless, rama de preview por PR, funciona igual desde Vercel y desde Cloudflare Workers. |
| Acceso a datos | **Drizzle ORM** + migraciones SQL versionadas | Tipado sin generador aparte; el SQL queda visible y revisable. |
| Autenticación | **Auth.js v5** (Google) + lista blanca + OTP de aprobación | Ver `AUTENTICACION.md`. |
| Correo | **Resend** | Para el OTP al dueño. Un solo remitente, dominio verificado. |
| Hosting | **Vercel** | Despliegue por push, preview por PR. |
| Borde | **Cloudflare** (DNS proxy, WAF, rate limiting, Turnstile) | Frena fuerza bruta antes de que toque la app. |
| Archivos (facturas) | **Cloudflare R2** | Guardar el PDF/XML original. Sin egress. Fase 5. |

## Estructura

```
app/
  (app)/                 vistas autenticadas: inicio, lista, menu, recetas, facturas, gastos
  (auth)/login/          Google + pantalla de código
  api/
    auth/[...nextauth]/  Auth.js
    acceso/solicitar/    crea la solicitud y manda el OTP al dueño
    acceso/verificar/    valida el código
    acceso/decidir/      links firmados aprobar/rechazar del correo
    facturas/parsear/    XML DIAN y PDF -> líneas
    ...
lib/
  db/                    esquema Drizzle + migraciones
  auth/                  configuración, allowlist, OTP, rate limit
  dominio/               lista de mercado, pronóstico, precios, parseo DIAN  (sin I/O, testeable)
  seguridad/             headers, validación zod, auditoría
components/
tests/
docs/
prototipo/               HTML de la Fase 1, referencia visual y de dominio
```

## Reglas de frontera

- **`lib/dominio/` no importa nada de red ni de base de datos.** Recibe datos, devuelve datos. Ahí viven las reglas que hoy están en el prototipo: generación de la lista, pronóstico, historial de precios, parseo de facturas. Es lo único que se prueba con pruebas unitarias puras.
- **Ningún componente cliente consulta la base directamente.** Server Component o Route Handler.
- **Toda entrada externa pasa por un esquema `zod`** en el borde del handler. Nada de `body as Tipo`.
- **Toda consulta filtra por hogar** (`household_id` de la sesión). No hay consulta sin ese filtro.

## Multi-hogar

La app se modela con `household` desde el día uno aunque hoy solo exista uno. Es una columna, no una funcionalidad: evita rehacer todas las consultas después y hace que el aislamiento de datos sea explícito.

## Renderizado y datos

- Lectura: Server Components con `fetch`/consulta directa, revalidación por etiqueta.
- Escritura: Server Actions para formularios simples; Route Handlers cuando hay archivos o un tercero.
- El estado compartido en vivo (varios en el mercado tildando la lista) llega en Fase 6 con polling cada 10 s; no hay WebSockets hasta que se demuestre que hacen falta.

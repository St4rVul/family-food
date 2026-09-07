# Mercado Sepúlveda

App familiar para planear el mercado, cocinar con lo que hay y saber en qué se va la plata.

- **Prototipo funcionando (Fase 1):** `prototipo/sepulveda.html` — React + Tailwind, sin backend, datos en el navegador o en el almacén del artifact.
- **Producto (Fase 2+):** Next.js + Postgres + Google Sign-In con lista blanca de correos, desplegado en Vercel detrás de Cloudflare.

## Documentación

| Documento | Qué contiene |
|---|---|
| [docs/ARQUITECTURA.md](docs/ARQUITECTURA.md) | Stack, decisiones, estructura de carpetas |
| [docs/MODELO-DATOS.md](docs/MODELO-DATOS.md) | Tablas, relaciones, reglas de integridad |
| [docs/AUTENTICACION.md](docs/AUTENTICACION.md) | Google Sign-In, lista blanca, OTP de aprobación al dueño |
| [docs/SEGURIDAD.md](docs/SEGURIDAD.md) | OWASP Top 10 2021 aplicado, controles y verificaciones |
| [docs/OPERACION.md](docs/OPERACION.md) | Variables de entorno, despliegue, respaldos, runbook |
| [docs/BACKLOG.md](docs/BACKLOG.md) | Fases, tareas y estado |
| [docs/DECISIONES.md](docs/DECISIONES.md) | Registro de decisiones (ADR) |

## Agentes

En `.claude/agents/` hay tres agentes con responsabilidad delimitada: `frontend`, `backend` y `seguridad`.
Se invocan con el nombre en una tarea, por ejemplo: *"usa el agente backend para implementar B-03"*.

## Reglas de la casa

1. Cada tarea del backlog tiene ID (`F2-B-03`). Los commits lo citan.
2. Nada se marca hecho sin una verificación que falle si se rompe.
3. El agente `seguridad` revisa todo lo que toque autenticación, sesiones, datos personales o dinero.

# Operación

## Entornos

| Entorno | Dónde | Base de datos |
|---|---|---|
| local | `npm run dev` | rama `dev` de Neon |
| preview | Vercel, por PR | rama efímera de Neon |
| producción | Vercel + Cloudflare | rama `main` de Neon |

## Variables de entorno

```
DATABASE_URL
AUTH_SECRET
AUTH_GOOGLE_ID
AUTH_GOOGLE_SECRET
OWNER_EMAIL
RESEND_API_KEY
ACCESS_DECISION_SECRET
TURNSTILE_SECRET_KEY
NEXT_PUBLIC_TURNSTILE_SITE_KEY
R2_ACCOUNT_ID / R2_ACCESS_KEY_ID / R2_SECRET_ACCESS_KEY / R2_BUCKET   # Fase 5
```

Se cargan en Vercel por entorno. Nunca en el repositorio. `.env.example` lista los nombres, jamás los valores.

## Despliegue

1. Push a una rama → preview automática en Vercel con su propia base de datos.
2. Migraciones: `npm run db:migrate` corre en el `build` y es idempotente.
3. Merge a `main` → producción.
4. Reversión: `vercel rollback`. Si la migración no es reversible, se escribe la migración de vuelta antes de desplegar.

## Cloudflare delante de Vercel

- DNS del dominio en Cloudflare, registro con proxy activo apuntando a Vercel.
- WAF: regla de rate limiting sobre `/api/acceso/*` (10 peticiones por minuto por IP) y sobre `/api/*` (100 por minuto).
- Turnstile con la clave del sitio en `/login`.
- Reglas de país solo si aparece abuso: la familia entra desde Colombia.

## Google OAuth

- Consola de Google Cloud → OAuth 2.0 Client ID tipo *Aplicación web*.
- Orígenes autorizados: el dominio de producción y `http://localhost:3000`.
- URI de redirección: `https://DOMINIO/api/auth/callback/google`.
- Pantalla de consentimiento en modo *External* + *Testing* con los correos de la familia, o *Internal* si hay Workspace.

## Correo (Resend)

- Dominio verificado con SPF, DKIM y DMARC. Sin esto, el correo del OTP se va a spam y el respaldo no sirve.
- Remitente `acceso@DOMINIO`. Un solo destinatario: `OWNER_EMAIL`.

## Respaldos

- Neon: recuperación a un punto en el tiempo, 7 días.
- Export semanal a R2 con retención de 8 semanas.
- Restauración probada una vez por trimestre; se anota la fecha en el registro del backlog.

## Runbook

**No llega el correo del OTP.** Revisar el panel de Resend (rebote o spam), luego SPF/DKIM. Mientras tanto, el dueño agrega el correo directo a la lista blanca desde su panel.

**Alguien no puede entrar con Google.** Verificar que el correo esté en `allowed_email` sin revocar, y que la cuenta de Google tenga el correo verificado. Revisar `audit_log` filtrando por ese correo.

**Se registró dos veces la misma factura.** El `cufe` único debería impedirlo. Si entró por PDF o manual (sin cufe), se borra la compra duplicada desde el panel; el historial de precios se recalcula solo.

**Gasto disparado en el pronóstico.** Suele ser una compra grande cargada con fecha equivocada. Revisar `purchase` del mes ordenado por total.

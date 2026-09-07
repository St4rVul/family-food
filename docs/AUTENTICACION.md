# Autenticación y control de acceso

Regla base: **la app no es pública**. Nadie crea cuenta solo. Existen dos caminos y ambos terminan en la lista blanca.

## Camino 1 — Google Sign-In con lista blanca

1. La persona entra a `/login` y pulsa *Continuar con Google*.
2. Google devuelve el perfil. En el callback `signIn` de Auth.js:
   - Se exige `email_verified === true`. Si no, se rechaza.
   - Se busca el correo en `allowed_email` (minúsculas, sin alias `+`, sin revocar).
   - **Si está**: se crea o actualiza el `member`, se abre sesión, se registra en `audit_log`.
   - **Si no está**: no se crea nada. Se dispara el Camino 2 y se muestra la pantalla neutra de "solicitud enviada".
3. La sesión se guarda en la tabla `session` (no solo JWT) para poder revocarla desde el perfil del dueño.

## Camino 2 — Aprobación con OTP al dueño (respaldo)

Pensado para el familiar que todavía no está en la lista, o cuando Google falla.

1. Se crea un `access_request` con: correo, IP, user-agent, país y ciudad (cabeceras `cf-ipcountry` / `x-vercel-ip-city`), y la hora en `America/Bogota`.
2. Se genera un código de **6 dígitos** con `crypto.randomInt`. **No se guarda el código**: se guarda `sha256(codigo + salt)` con un salt por solicitud.
3. Se envía un correo **al dueño** (`OWNER_EMAIL`), nunca al solicitante, con:
   - quién lo pide (correo y nombre de Google si lo hubo),
   - desde dónde (IP, ciudad, país, dispositivo/navegador),
   - cuándo,
   - el código de 6 dígitos,
   - dos enlaces firmados: **Aprobar y agregar a la lista** / **Rechazar y bloquear**.
4. Al solicitante se le muestra siempre lo mismo: *"Le avisamos a Daniel. Pídele el código para entrar."* No se revela si el correo existe ni si está en la lista.
5. El solicitante escribe el código. Si coincide, se crea el `member` con rol `menor` por defecto y se marca la solicitud como `usada`.

### Reglas del código

| Regla | Valor |
|---|---|
| Longitud | 6 dígitos, generados con CSPRNG |
| Vigencia | 10 minutos |
| Intentos | 5; al sexto la solicitud queda `expirada` |
| Uso | único; se invalida al usarse o al decidirse |
| Comparación | en tiempo constante sobre el hash |
| Solicitudes por correo | 3 por hora |
| Solicitudes por IP | 5 por hora, 20 por día |
| Turnstile | obligatorio antes de crear la solicitud |

### Enlaces de decisión del correo

- Token = HMAC-SHA256 sobre `request_id + accion + expiración`, con `ACCESS_DECISION_SECRET`.
- Vigencia 30 minutos, un solo uso, verificación en tiempo constante.
- El enlace lleva a una página que **pide confirmación explícita** antes de actuar: un prefetch del cliente de correo no puede aprobar a nadie.
- *Rechazar y bloquear* deja el correo en una lista de bloqueo: sus solicitudes futuras no generan correo, solo se registran.

## Roles

| Acción | dueño | adulto | menor |
|---|:--:|:--:|:--:|
| Ver y editar la lista y el carrito | sí | sí | sí |
| Ver recetas y menú | sí | sí | sí |
| Ver montos, facturas y gastos | sí | sí | **no** |
| Registrar compras | sí | sí | no |
| Administrar lista blanca y sesiones | sí | no | no |
| Ver auditoría | sí | no | no |

La autorización se resuelve **en el servidor en cada handler**. Que un botón no se pinte no es un control de acceso.

## Sesiones

- Cookies `HttpOnly`, `Secure`, `SameSite=Lax`, `__Host-` como prefijo.
- Duración 30 días con renovación deslizante; se invalida al cambiar el rol o al revocar el correo.
- El dueño ve las sesiones activas (dispositivo, IP, último uso) y puede cerrarlas.

## Variables de entorno

```
AUTH_SECRET                 # Auth.js
AUTH_GOOGLE_ID
AUTH_GOOGLE_SECRET
OWNER_EMAIL                 # a quién le llega el OTP
RESEND_API_KEY
ACCESS_DECISION_SECRET      # HMAC de los enlaces aprobar/rechazar
TURNSTILE_SECRET_KEY
NEXT_PUBLIC_TURNSTILE_SITE_KEY
DATABASE_URL
```

## Qué se prueba

- Correo fuera de la lista blanca **no** crea `member` (ni por Google ni por OTP sin aprobar).
- Código vencido, ya usado, o con 5 fallos previos → rechazado.
- Código de una solicitud no sirve en otra.
- Enlace de decisión con firma alterada o vencida → rechazado.
- La respuesta al solicitante es idéntica exista o no el correo (sin enumeración de usuarios).
- Un `menor` que pide `/api/gastos` recibe 403 aunque la UI no muestre el enlace.

# Seguridad — OWASP Top 10 (2021) aplicado

Cada punto lista el riesgo concreto en **esta** app, el control y cómo se verifica. El agente `seguridad` revisa contra este documento.

## A01 · Pérdida de control de acceso

- **Riesgo:** un familiar ve las facturas y montos que no le corresponden; alguien manipula un `id` en la URL y lee datos de otro hogar.
- **Controles:** autorización en cada Route Handler y Server Action, nunca solo en la UI. Toda consulta filtra por `household_id` de la sesión. Los `id` son UUID v4, no correlativos. Los roles se resuelven en el servidor a partir de `session → member.rol`.
- **Verificación:** prueba por cada endpoint con las tres combinaciones (sin sesión, rol insuficiente, otro hogar). Ninguna debe devolver 200.

## A02 · Fallas criptográficas

- **Riesgo:** el OTP guardado en claro; secretos en el repo; tráfico sin cifrar.
- **Controles:** el OTP se guarda como `sha256(codigo + salt)` con salt por registro; comparación en tiempo constante. Secretos solo por variables de entorno. HSTS y solo HTTPS. Los tokens de los enlaces del correo van firmados con HMAC-SHA256 y expiran.
- **Verificación:** `gitleaks` en CI. Revisión de que ningún `select` de OTP devuelva el código en claro (no existe la columna).

## A03 · Inyección

- **Riesgo:** SQL por concatenación; XSS al pintar la descripción que viene de la factura; XXE al leer el XML de la DIAN.
- **Controles:** Drizzle con consultas parametrizadas; prohibido `sql.raw` con datos de usuario. React escapa por defecto y **no se usa `dangerouslySetInnerHTML`**. El XML se parsea con entidades externas y DTD deshabilitadas; el archivo se lee con límite de tamaño y de profundidad. Todo cuerpo de petición pasa por `zod`.
- **Verificación:** prueba con un XML con `<!ENTITY` externo → debe fallar sin hacer ninguna petición de red; prueba con `descripcion` = `<img onerror>` → se pinta como texto.

## A04 · Diseño inseguro

- **Riesgo:** el flujo de OTP es la puerta de entrada; mal diseñado, es la vulnerabilidad principal de la app.
- **Controles:** el OTP llega **al dueño**, no al solicitante. Límite de intentos, vigencia corta, un solo uso, Turnstile antes de generar la solicitud, rate limit por correo e IP. Rol mínimo (`menor`) al entrar por este camino. Respuesta idéntica exista o no el correo.
- **Verificación:** ver la lista de pruebas de `AUTENTICACION.md`.

## A05 · Configuración incorrecta

- **Controles:** cabeceras en `next.config`: `Content-Security-Policy` con nonce (sin `unsafe-inline`), `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` restrictiva, `X-Frame-Options: DENY`. Sin `/api` de depuración en producción. Errores genéricos al cliente, detalle solo en el log del servidor. Base de datos sin acceso público, solo desde Vercel y las IP del dueño.
- **Verificación:** revisión de cabeceras en cada despliegue (`curl -I`), y que una excepción no devuelva stack trace.

## A06 · Componentes vulnerables

- **Controles:** `npm audit` y Dependabot en CI; versiones fijadas; no se agrega una dependencia sin entrada en `DECISIONES.md`. `pdf.js` se ejecuta **en el navegador** con el worker aislado, nunca en el servidor con archivos de terceros.
- **Verificación:** CI falla con vulnerabilidad alta o crítica.

## A07 · Fallas de identificación y autenticación

- **Controles:** solo Google (sin contraseñas propias) más el OTP de respaldo. Se exige `email_verified`. Sesiones en base de datos, revocables. Cookies `HttpOnly`/`Secure`/`SameSite=Lax` con prefijo `__Host-`. Rotación del identificador de sesión al iniciar sesión. Cierre de sesión invalida el registro en servidor, no solo la cookie.
- **Verificación:** revocar en la tabla `session` corta el acceso en la siguiente petición.

## A08 · Fallas de integridad de software y datos

- **Controles:** `package-lock.json` versionado y CI con `npm ci`. Los enlaces de aprobación van firmados. Los archivos subidos se validan por tipo real y tamaño (XML/PDF, máximo 10 MB), se guardan con nombre generado y **nunca se sirven desde el mismo origen de la app**.
- **Verificación:** subir un `.pdf` que en realidad es HTML → rechazado.

## A09 · Fallas de registro y monitoreo

- **Controles:** `audit_log` append-only para ingresos, solicitudes de acceso, decisiones, cambios de rol, creación y borrado de compras. Nunca se registran códigos OTP, tokens ni cookies. Alerta por correo al dueño ante: 3 solicitudes fallidas de un mismo correo, o una decisión de aprobación.
- **Verificación:** prueba de que un intento fallido de OTP queda registrado y que el código no aparece en el registro.

## A10 · Falsificación de solicitudes del lado del servidor (SSRF)

- **Riesgo:** el enlace de receta o el XML de la factura hacen que el servidor pida una URL controlada por un tercero.
- **Controles:** el servidor **no** descarga URL suministradas por el usuario. Los enlaces de recetas se guardan como texto y se abren en el cliente con `rel="noopener noreferrer"`. Parseo de XML sin resolución de entidades externas ni DTD.
- **Verificación:** no debe existir ningún `fetch` del servidor cuyo destino provenga de datos de usuario.

## Más allá del Top 10

- **Datos personales:** solo correo y nombre. Sin teléfono, sin dirección, sin datos de pago. Los archivos de factura llevan NIT y dirección del comercio: se guardan cifrados en reposo (R2) y no se exponen por URL pública.
- **Rate limiting general:** 100 peticiones por minuto por sesión; `/api/acceso/*` mucho más estricto (ver `AUTENTICACION.md`). Cloudflare aplica la primera capa antes de llegar a Vercel.
- **Respaldos:** Neon con recuperación a un punto en el tiempo de 7 días; export semanal a R2. Restauración probada una vez por trimestre.
- **Borrado:** el dueño puede exportar y borrar todos los datos del hogar desde la app.

## Lista de verificación previa a producción

- [ ] Cabeceras de seguridad presentes y CSP sin `unsafe-inline`
- [ ] Sin secretos en el repositorio (`gitleaks` limpio)
- [ ] Todos los endpoints con prueba de autorización negativa
- [ ] Rate limit activo en `/api/acceso/*` y verificado
- [ ] Turnstile activo en `/login`
- [ ] `audit_log` escribiendo y sin datos sensibles
- [ ] Respaldo restaurado con éxito al menos una vez
- [ ] Revisión del agente `seguridad` sin hallazgos abiertos

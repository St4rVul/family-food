# Registro de decisiones

Formato: contexto → decisión → consecuencia. Se agrega una entrada por cada elección que sea costosa de revertir o que agregue una dependencia.

## D-001 · Un solo repositorio con Next.js en vez de front y API separados
**2026-09-07.** Una familia, seis vistas, un despliegue. Separar en dos servicios duplicaría el despliegue, la autenticación y el CORS sin ganar nada. Consecuencia: si algún día hay app nativa, la API ya está en `app/api` y se reutiliza.

## D-002 · Postgres en Neon, no SQLite ni Firebase
**2026-09-07.** Hay datos relacionales de verdad (compras, líneas, productos, historial de precios) y consultas de agregación para los gastos. Neon da rama por preview y funciona igual desde Vercel y desde Cloudflare. Consecuencia: hay que manejar migraciones; se versionan en `lib/db/migrations`.

## D-003 · Dinero en enteros
**2026-09-07.** Pesos colombianos sin decimales. `float` en dinero produce diferencias de un peso que después nadie explica. Consecuencia: todo cálculo redondea explícitamente al guardar.

## D-004 · El OTP llega al dueño, no al solicitante
**2026-09-07.** Es lo que pidió la casa y además es más seguro: convierte un factor de posesión de un tercero en una aprobación humana explícita. Consecuencia: el dueño es un punto único; si no está disponible, nadie entra. Se mitiga con la lista blanca, que es el camino normal.

## D-005 · Rol `menor` por defecto al entrar por OTP
**2026-09-07.** Quien entra por el camino de respaldo no ve montos ni facturas hasta que el dueño le suba el rol. Consecuencia: un adulto nuevo necesita dos pasos; es un paso más, no un bloqueo.

## D-006 · El XML de la DIAN por encima del PDF
**2026-09-07.** El XML trae las líneas exactas y el `cufe`. El PDF es heurística y muchos son escaneados. Consecuencia: la UI empuja el XML pero nunca obliga; la ruta manual siempre existe.

## D-007 · `household_id` desde el día uno
**2026-09-07.** Es una columna, no una funcionalidad. Agregarla después obliga a reescribir todas las consultas y a auditar el aislamiento de datos con la app ya en uso.

## D-008 · Sin librería de componentes
**2026-09-07.** La app tiene unos diez patrones visuales y ya están escritos en el prototipo. Una librería agrega peso, versiones y un estilo que después hay que pelear. Consecuencia: los componentes se mantienen a mano en `components/`.

## D-009 · Un precio solo existe si salió de una factura
**2026-09-07.** Los precios de referencia sembrados a mano daban totales creíbles pero falsos, y falsos son peores que ausentes: nadie los revisa. Ahora un producto sin historial muestra "—" y el total de la lista dice cuántos productos aún no tienen precio. Consecuencia: las primeras semanas la app muestra pocos montos; a la tercera factura ya calcula sola.

## D-010 · Íconos por categoría ahora, fotos reales después
**2026-09-07.** Lo que hace fácil escoger es reconocer el producto de un vistazo. Un ícono dibujado por categoría se logra hoy, sin subir nada, sin peso y funciona sin señal dentro del supermercado. Las fotos por producto necesitan almacenamiento, recorte y a alguien que las suba: queda en `F5-F-02`, y el ícono sigue como respaldo cuando no haya foto.

## D-011 · Se arranca con el subdominio de Vercel, no con dominio propio
**2026-09-07.** `*.vercel.app` sirve para Google OAuth, para el OTP de Resend (el remitente de prueba escribe al dueño de la cuenta, que es justo a quien va el código) y para Turnstile. Comprar dominio solo cambia la estética y se puede hacer después sin tocar código.

## D-012 · Auth.js, no Neon Auth
**2026-09-07.** Neon ofrece autenticación integrada con sincronización de usuarios a la base. Suena a menos piezas, pero nuestro requisito no es "login con Google": es login con Google **filtrado por lista blanca**, con un camino de respaldo donde el código va al dueño y no al solicitante. Eso es un gancho a la medida en el callback `signIn`, que Auth.js expone directo y sin atarnos a un producto en beta. Consecuencia: la tabla de usuarios la manejamos nosotros; `member` y `allowed_email` son nuestras.

## D-013 · Base de datos en us-east-1, la misma región de Vercel
**2026-09-07.** La latencia que importa no es Colombia→base, es función→base: ocurre varias veces por petición. Vercel Hobby corre en `iad1` (N. Virginia); la base va ahí mismo.

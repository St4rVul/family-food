# Backlog

Estado: `[ ]` pendiente · `[~]` en curso · `[x]` hecho. Cada tarea tiene ID `FASE-ÁREA-NN`.
Áreas: **F** front, **B** back, **S** seguridad, **O** operación, **D** diseño/dominio.

---

## Fase 1 — Prototipo sin backend  ✅ 2026-09-07

- [x] `F1-D-01` Catálogo de productos reales de la casa, con frecuencia semanal/mensual y precio de referencia
- [x] `F1-D-02` 36 recetas con ingredientes, pasos, tiempo y costo por porción
- [x] `F1-D-03` Generación de la lista: menú de la semana + recurrentes + mensuales por vencer
- [x] `F1-D-04` Historial de precios y alertas de variación
- [x] `F1-D-05` Pronóstico de gasto por regresión sobre meses cerrados
- [x] `F1-F-01` React + Tailwind, iconos de trazo, tema claro y oscuro, móvil y escritorio
- [x] `F1-F-02` Lectura de factura: XML DIAN (UBL, incluido `AttachedDocument`) y PDF con capa de texto
- [x] `F1-F-03` Registro de compra solo con el total, para reconstruir meses pasados con el extracto bancario
- [x] `F1-F-04` Categorías: supermercado, compras aparte, aseo, perros, medicamentos, otros
- [x] `F1-F-05` Íconos ilustrados por categoría de ingrediente y selector visual en cuadrícula
- [x] `F1-D-06` Fuera los precios inventados: un precio solo existe si salió de una factura

**Entregado:** `prototipo/sepulveda.html`

---

## Fase 0 — Cuentas y accesos  ← **te toca a ti**

Ver `TODO.md` en la raíz. Bloquea las fases 2 y 3.

- [ ] `F0-O-01` Repositorio GitHub privado
- [ ] `F0-O-02` Proyecto en Vercel conectado al repo, con su dirección `*.vercel.app`
- [ ] `F0-O-03` Base de datos en Neon (us-east-1)
- [ ] `F0-O-04` Credenciales OAuth de Google (localhost + producción)
- [ ] `F0-O-05` Cuenta de Resend con danielsanmarquez84@gmail.com
- [ ] `F0-O-06` Turnstile en Cloudflare *(opcional, se puede dejar para el final)*
- [ ] `F0-O-07` Lista de correos de la familia con su rol

---

## Fase 2 — Fundación del producto

- [ ] `F2-O-01` Repositorio Next.js 15 + TypeScript + Tailwind, ESLint, Prettier, Vitest
- [ ] `F2-O-02` Neon: base de datos, rama de desarrollo, `DATABASE_URL` en Vercel
- [ ] `F2-B-01` Esquema Drizzle completo de `MODELO-DATOS.md` + primera migración
- [ ] `F2-B-02` Semillas: hogar Sepúlveda, catálogo y recetas migrados del prototipo
- [ ] `F2-D-01` Mover `lib/dominio/` desde el prototipo (lista, pronóstico, precios, parseo DIAN) con pruebas unitarias
- [ ] `F2-O-03` CI: `npm ci`, typecheck, pruebas, `npm audit`, `gitleaks`
- [ ] `F2-O-04` Despliegue a Vercel con preview por PR

## Fase 3 — Acceso

- [ ] `F3-B-01` Auth.js v5 con Google, sesiones en base de datos
- [ ] `F3-B-02` Lista blanca: `allowed_email` y bloqueo en el callback `signIn`
- [ ] `F3-B-03` Solicitud de acceso: registro de IP, ciudad, país, dispositivo
- [ ] `F3-B-04` OTP al dueño por Resend, con enlaces firmados de aprobar/rechazar
- [ ] `F3-B-05` Verificación del código: vigencia, intentos, uso único, tiempo constante
- [ ] `F3-S-01` Rate limit por correo e IP en `/api/acceso/*`
- [ ] `F3-S-02` Turnstile en `/login`
- [ ] `F3-B-06` Roles `dueno`/`adulto`/`menor` y comprobación en cada handler
- [ ] `F3-F-01` Pantallas: login, "solicitud enviada", ingreso de código, acceso denegado
- [ ] `F3-F-02` Panel del dueño: lista blanca, solicitudes, sesiones activas, auditoría
- [ ] `F3-S-03` Revisión completa del agente `seguridad` sobre el flujo de acceso

## Fase 4 — La app

- [ ] `F4-F-01` Migrar las seis vistas del prototipo a Server Components
- [ ] `F4-B-01` API de productos, recetas, menú, lista y compras
- [ ] `F4-B-02` Carrito compartido: cualquiera agrega, se ve quién pidió qué
- [ ] `F4-F-02` Perfiles: avatar, "lo que pidió cada uno" en la lista
- [ ] `F4-B-03` Pendientes por comprar con categoría, y su paso a compra registrada
- [ ] `F4-F-03` Lista del sábado en modo mercado: pantalla grande, un toque, sin distracciones

## Fase 5 — Facturas de verdad

- [ ] `F5-B-01` Subida a R2 del XML/PDF original, cifrado en reposo
- [ ] `F5-B-02` Parseo en el servidor con validación estricta y sin entidades externas
- [ ] `F5-B-03` `cufe` único: no se registra dos veces la misma factura
- [ ] `F5-B-04` Emparejamiento aprendido: recordar que "PECHUGA BANDEJA" es `pollo` para esa tienda
- [ ] `F5-F-01` Revisión y corrección de líneas antes de guardar
- [ ] `F5-D-01` Pronóstico mejorado: estacionalidad y tendencia por producto
- [ ] `F5-F-02` Fotos reales de productos: subida a R2, recorte cuadrado, respaldo al ícono de categoría

## Fase 6 — Vivir con ella

- [ ] `F6-F-01` PWA instalable, funciona sin señal dentro del supermercado
- [ ] `F6-F-02` Sincronización de la lista entre quienes están mercando (sondeo cada 10 s)
- [ ] `F6-O-01` Cloudflare delante de Vercel: DNS, WAF, rate limiting
- [ ] `F6-O-02` Respaldo semanal a R2 y prueba de restauración
- [ ] `F6-F-03` Exportar e importar todos los datos del hogar
- [ ] `F6-S-01` Repaso de la lista de verificación previa a producción

---

## Registro

| Fecha | Tarea | Nota |
|---|---|---|
| 2026-09-07 | Fase 1 completa | Prototipo publicado; el HTML queda como referencia visual y de dominio |
| 2026-09-07 | Documentación inicial | Arquitectura, modelo de datos, autenticación, seguridad, backlog y agentes |
| 2026-09-07 | `F1-F-05`, `F1-D-06` | Íconos por categoría y selector visual; se quitaron los precios de referencia |
| 2026-09-07 | Fase 0 abierta | `TODO.md` con los pasos de cuentas |

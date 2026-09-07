# Instrucciones del proyecto

App de mercado y gastos para una familia de 4 en Colombia. Español en todo: UI, commits, documentación, comentarios.

## Contexto de dominio

- Se merca los **sábados**. Hay productos semanales y otros mensuales.
- Pesos colombianos, sin decimales. Unidades reales: `lb`, `kg`, `u`, `l`, `manojo`, `paca`.
- Facturación electrónica DIAN: el **XML** (UBL 2.1, a veces envuelto en `AttachedDocument` con el `Invoice` dentro de un CDATA) es la fuente exacta. El PDF es el respaldo. Muchos PDF son escaneados y no tienen capa de texto: siempre debe existir la ruta manual.
- Categorías de gasto: supermercado, compras aparte, aseo, perros, medicamentos, otros.

## Reglas técnicas

- Stack fijado en `docs/ARQUITECTURA.md`. No introducir dependencias nuevas sin una línea en `docs/DECISIONES.md`.
- La solución más corta que funcione. Sin abstracciones especulativas, sin capas para un solo caso.
- Toda lógica no trivial (dinero, fechas, parseo de facturas, permisos) deja una prueba que falle si se rompe.
- Dinero en **enteros** (pesos). Nunca `float` para valores monetarios en la base de datos.
- Fechas en `YYYY-MM-DD` y zona `America/Bogota`. La semana va de **sábado a viernes**.
- Nada de secretos en el repo. Todo por variables de entorno.

## Flujo de trabajo

1. Antes de codear, ubicar la tarea en `docs/BACKLOG.md` y trabajar contra su ID.
2. Al terminar: marcar la tarea, anotar la fecha y agregar la entrada al registro al final del backlog.
3. Cambios que tocan auth, sesiones, datos personales o dinero pasan por el agente `seguridad` antes de darse por hechos.

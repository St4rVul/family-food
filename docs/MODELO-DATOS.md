# Modelo de datos

Postgres. Dinero en `integer` (pesos colombianos, sin decimales). Fechas de calendario en `date`; marcas de tiempo en `timestamptz`. Toda tabla de negocio lleva `household_id`.

## Tablas

### Identidad y acceso

**`household`** — el hogar. `id`, `nombre`, `creado_en`.

**`member`** — perfil de cada persona. `id`, `household_id`, `email` (único, minúsculas), `nombre`, `avatar_url`, `rol` (`dueno` | `adulto` | `menor`), `activo`, `creado_en`, `ultimo_ingreso`.
- `dueno`: administra la lista blanca y ve la auditoría. Solo uno.
- `adulto`: todo menos administrar accesos.
- `menor`: agrega al carrito y ve recetas; no ve montos ni facturas.

**`allowed_email`** — lista blanca. `id`, `household_id`, `email`, `rol_asignado`, `agregado_por`, `creado_en`, `revocado_en`.
Un correo fuera de esta tabla **no crea cuenta**: solo genera una solicitud.

**`access_request`** — intento de ingreso que necesita aprobación. `id`, `email`, `ip`, `user_agent`, `pais`, `ciudad`, `codigo_hash`, `codigo_salt`, `intentos`, `estado` (`pendiente`|`aprobada`|`rechazada`|`expirada`|`usada`), `expira_en`, `creado_en`, `decidido_en`, `decidido_por`.

**`session`** — sesiones en base de datos (no solo JWT) para poder revocar. `id`, `member_id`, `expira_en`, `ip`, `user_agent`, `revocada_en`.

**`audit_log`** — `id`, `household_id`, `actor_id`, `accion`, `entidad`, `entidad_id`, `datos` (jsonb, sin datos sensibles), `ip`, `creado_en`. Append-only: sin `UPDATE` ni `DELETE` desde la app.

### Dominio

**`product`** — catálogo. `id`, `household_id`, `nombre`, `categoria`, `unidad`, `frecuencia` (`sem`|`mes`|`ocas`), `cantidad_tipica` (numeric), `precio_referencia` (int), `activo`.

**`recipe`** — `id`, `household_id`, `nombre`, `comida` (`desayuno`|`almuerzo`|`cena`), `minutos`, `porciones`, `pasos` (text[]), `etiquetas` (text[]), `nota`, `enlace`, `creado_por`.

**`recipe_ingredient`** — `recipe_id`, `product_id`, `cantidad` (numeric). PK compuesta.

**`meal_plan`** — `household_id`, `fecha` (date), `comida`, `recipe_id`. PK compuesta. La semana es sábado→viernes.

**`cart_item`** — el carrito compartido: lo que cualquiera agrega desde su perfil. `id`, `household_id`, `product_id` (nullable), `texto_libre`, `cantidad`, `categoria` (`supermercado`|`aparte`|`aseo`|`perros`|`medicamentos`|`otros`), `nota`, `agregado_por`, `estado` (`pendiente`|`comprado`|`descartado`), `semana` (date del sábado), `comprado_en`, `purchase_id`.
- `product_id` nulo + `texto_libre` permite pedir algo que no está en el catálogo sin bloquear a nadie.
- Es la tabla que une "pendientes por comprar" con la lista del sábado: la lista generada = ingredientes del menú + recurrentes por vencer + `cart_item` pendientes.

**`purchase`** — una compra. `id`, `household_id`, `fecha`, `lugar`, `categoria`, `total` (int), `origen` (`xml`|`pdf`|`manual`|`lista`), `archivo_key` (R2), `cufe` (único por hogar, nullable), `registrado_por`, `creado_en`.
- `cufe`: el identificador de la factura electrónica DIAN. Único → **evita registrar dos veces la misma factura**.

**`purchase_item`** — `id`, `purchase_id`, `product_id` (nullable), `descripcion_original`, `cantidad` (numeric), `valor` (int).
- `descripcion_original` se guarda siempre: es lo que permite mejorar el emparejamiento después sin volver a leer la factura.

**`price_point`** — vista materializada o tabla derivada: `product_id`, `fecha`, `precio_unitario`. Se alimenta de `purchase_item`. Es la base del historial de precios, las alertas y el pronóstico.

## Índices

- `member(email)` único; `allowed_email(household_id, email)` único parcial `WHERE revocado_en IS NULL`.
- `access_request(email, creado_en)` y `access_request(ip, creado_en)` para el rate limit.
- `purchase(household_id, fecha DESC)`, `purchase_item(product_id)`, `cart_item(household_id, estado, semana)`.
- `purchase(household_id, cufe)` único parcial `WHERE cufe IS NOT NULL`.

## Integridad

- `CHECK (total >= 0)`, `CHECK (valor >= 0)`, `CHECK (cantidad > 0)`.
- `ON DELETE CASCADE` de `purchase` a `purchase_item` y de `recipe` a `recipe_ingredient`.
- `product` no se borra si tiene historial: se marca `activo = false`.
- Nada de borrado físico en `purchase`: se conserva para la historia de precios.

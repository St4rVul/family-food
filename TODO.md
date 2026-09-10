# TODO de Daniel — cuentas y accesos

Esto es lo único que no puedo hacer yo: crear cuentas a tu nombre. Todo lo demás lo hago cuando esto esté listo.

**Regla de oro:** los valores marcados 🔒 **no me los pegues en el chat**. Van directo a `.env.local` o al panel de Vercel. A mí solo dime "listo".
Los marcados 📋 sí me los puedes pasar, no son secretos.

Tiempo total estimado: **50–70 minutos**. Se puede hacer en dos ratos.

---

## 1 · GitHub (10 min)

- [x] Repositorio creado: **github.com/St4rVul/family-food**
- [x] `gh` instalado y autenticado como St4rVul
- [x] Primer push hecho (rama `main`)
- [x] 📋 Usuario de GitHub: **St4rVul**

> Yo hago: `git init`, primer commit, `git remote add`, push.

---

## 2 · Vercel (10 min)

- [ ] Entrar a https://vercel.com y **registrarse con la cuenta de GitHub** (así el despliegue queda conectado solo).
- [ ] Plan **Hobby** (gratis). No hace falta tarjeta.
- [ ] Dejar que Vercel vea el repo `family-food` (Install → seleccionar solo ese repositorio).
- [ ] 📋 Pasarme el nombre del proyecto que quede.

### Si el correo de Vercel no es el tuyo

Es normal. GitHub entrega un correo `...@users.noreply.github.com` cuando tienes activado *Keep my email addresses private*, y Vercel se queda con ese.

- [ ] Comprobar en Vercel → Settings → **Authentication** que la cuenta conectada sea **St4rVul**.
      - Si dice St4rVul → todo bien, sigue. El correo de Vercel no lo usa la app para nada.
      - Si dice otro usuario → cerrar sesión en Vercel **y en github.com**, entrar a GitHub como St4rVul y volver a intentar.
- [ ] (Opcional, cosmético) Vercel → Settings → General → Email → cambiarlo a danielsanmarquez84@gmail.com.

> El correo que recibe los OTP se configura aparte, en `OWNER_EMAIL`. No tiene nada que ver con el de Vercel ni con el de GitHub.

### Si te aparece la cuenta vieja de la empresa

Vercel enlaza cuentas **por correo**. Si ese correo de empresa está entre los verificados de tu GitHub, Vercel te mete a la cuenta vieja. Mira el selector de scope arriba a la izquierda:

**A · Sale un Team con el nombre de la empresa** — el caso más común y el más fácil:
- [ ] Clic en el Team → Settings → Members → tu fila → **Leave Team**. Vuelves a *Personal Account* y ya.

**B · Tu cuenta personal tiene el correo de la empresa:**
- [ ] Settings → General → Email → cambiarlo a danielsanmarquez84@gmail.com y confirmar. Misma cuenta, nada se pierde.

**C · Es otra cuenta de Vercel distinta** — desenredar en este orden:
- [ ] Settings → Authentication → **agregar primero** otro método de acceso (Email o Passkey)
      ⚠️ Vercel no deja quitar GitHub si es el único login
- [ ] Disconnect GitHub → cerrar sesión
- [ ] github.com/settings/applications → Authorized OAuth Apps → Vercel → **Revoke**
- [ ] github.com/settings/installations → si aparece Vercel → **Uninstall**
- [ ] vercel.com → Sign Up con el Gmail → luego Settings → Authentication → Connect GitHub (St4rVul)

**Opción nuclear:** Settings → General → abajo → *Delete Account*. Solo si esa cuenta no tiene proyectos de la empresa desplegados.

> **Esto no bloquea el proyecto.** Vercel solo despliega. Los pasos 3 y 4 se pueden hacer ya, y la app corre completa en local con `npm run dev` sin tocar Vercel.

### Sobre el dominio

No necesitas comprar nada todavía. Vercel regala un subdominio y funciona para **todo**, incluido Google y el correo del OTP:

```
mercado-sepulveda.vercel.app
```

- [x] Proyecto creado en el team **Star** (Hobby). Dirección generada: `family-food-seven-lilac.vercel.app`
- [ ] ⚠️ **Decidir la dirección definitiva antes de configurar Google.** El `*.vercel.app` generado no se edita, se agrega otro:
      - Settings → **Domains** → *Add Domain* → `mercado-sepulveda.vercel.app` → marcarlo como Production; **o**
      - Settings → **General** → *Project Name* → `mercado-sepulveda` → Vercel regenera el dominio.
      - Si pelea, deja `family-food-seven-lilac.vercel.app`: funciona igual, solo es feo. Lo que **no** se puede es cambiarla después de configurar Google.
- [ ] 📋 Confirmarme la dirección final.
- [ ] Application Preset: **Other** (no Create React App). El repo trae `vercel.json` con la configuración.

> Un dominio propio (`sepulveda.co`, ~$40.000/año) solo cambia la estética. Se puede agregar después sin tocar código: si quieres, se compra en Cloudflare y se apunta a Vercel. **Lo dejamos para el final**, no bloquea nada.

---

## 3 · Neon — base de datos (10 min)

- [ ] Crear cuenta en https://neon.tech (también con GitHub).
- [ ] **New Project**:
      - Nombre: `family-food`
      - Región: **AWS US East 1 (N. Virginia)** ← la misma donde Vercel corre las funciones (`iad1`)
      - Postgres 17
      - **Object storage: OFF** (es para los PDF de facturas, Fase 5)
      - **Neon Auth: OFF** (vamos con Auth.js; dos sistemas de login es problema seguro)
      - Functions y AI gateway: OFF
- [x] Proyecto creado · project-id `divine-water-82655296`
- [x] Región: **US East 2 (Ohio)**. Virginia no estaba disponible; la diferencia son ~15 ms por consulta y no se siente.
- [ ] Confirmar que *Object storage* y *Neon Auth* quedaron **apagados**
- [x] Cadena de conexión copiada y guardada como `DATABASE_URL`
- [ ] Confirmar que está en **Vercel → Settings → Environment Variables**, marcada para Production, Preview y Development
      > Los 7 pasos del CLI que sugiere Neon (`neon skills`, `neon mcp`, `neon deploy`…) **no se necesitan**: las migraciones van con Drizzle y el despliegue con Vercel.
- [ ] 🔒 Guardarla; va a ser `DATABASE_URL`. **No la pegues en el chat.**
- [ ] 📋 Confirmarme: proyecto creado y región.

> Capa gratis: 0.5 GB. Años de mercado de la familia caben de sobra.

---

## 4 · Google OAuth (15 min — el más enredado)

Necesitas la dirección del paso 2 antes de empezar.

- [ ] Ir a https://console.cloud.google.com → **Nuevo proyecto** → nombre `Mercado Sepulveda`.
- [ ] Menú → **APIs y servicios** → **Pantalla de consentimiento de OAuth**:
      - Tipo: **Externo**
      - Nombre de la app: `Mercado Sepúlveda`
      - Correo de asistencia: el tuyo
      - Datos de contacto del desarrollador: el tuyo
      - **Guardar y continuar** en todas las pantallas (no toques permisos)
      - En **Usuarios de prueba** → agregar los correos de la familia (hasta 100)
- [ ] Menú → **Credenciales** → **Crear credenciales** → **ID de cliente de OAuth**:
      - Tipo: **Aplicación web**
      - Nombre: `web`
      - **Orígenes autorizados de JavaScript:**
        ```
        http://localhost:3000
        https://family-food-seven-lilac.vercel.app
        ```
      - **URI de redireccionamiento autorizados:**
        ```
        http://localhost:3000/api/auth/callback/google
        https://family-food-seven-lilac.vercel.app/api/auth/callback/google
        ```

> Las previsualizaciones de Vercel tienen URL aleatoria y Google no acepta comodines: el login con Google sirve en local y en producción, no en previews. Es normal.
- [ ] 📋 Pasarme el **Client ID** (termina en `.apps.googleusercontent.com`, no es secreto).
- [ ] 🔒 Guardar el **Client secret** aparte.

> Se queda en modo *Prueba*: no hay que pedirle verificación a Google. Los correos de la familia van en *Usuarios de prueba* y con eso basta.

---

## 4b · Secretos que generas tú (2 min)

Estos dos no vienen de ningún panel. Son llaves nuestras.

- [ ] Correr **dos veces** en tu terminal y guardar cada resultado por aparte:
      ```
      openssl rand -base64 33
      ```
- [ ] 🔒 El primero → `AUTH_SECRET` (firma la cookie de sesión; el mismo en local y en Vercel)
- [ ] 🔒 El segundo → `ACCESS_DECISION_SECRET` (firma los enlaces Aprobar/Rechazar del correo)

> Tienen que ser distintos entre sí. Si `AUTH_SECRET` cambia después, se cierran todas las sesiones abiertas — no se pierde nada más.

---

## 5 · Resend — correo del OTP (10 min)

- [ ] Crear cuenta en https://resend.com con **danielsanmarquez84@gmail.com**
      ⚠️ Tiene que ser ese correo: es el que va a recibir los códigos.
- [ ] **API Keys** → Create API Key → permiso *Sending access*.
- [ ] 🔒 Guardarla como `RESEND_API_KEY`.
- [ ] 📋 Confirmarme: cuenta creada con ese correo.

> **No necesitas dominio propio todavía.** El remitente de prueba `onboarding@resend.dev` solo puede escribirle al correo dueño de la cuenta — que es exactamente a donde va el OTP. Cuando compres dominio, se verifica (SPF/DKIM/DMARC) y ya puede escribirle a más gente.

---

## 6 · Cloudflare (5 min — opcional, se puede dejar para después)

Solo tiene sentido cuando haya dominio propio.

- [ ] Cuenta gratis en https://cloudflare.com
- [ ] **Turnstile** → Add site → 📋 pasarme la **Site key** · 🔒 guardar la **Secret key**
      (Turnstile sí funciona desde ya en `*.vercel.app`.)

---

## 7 · Cuando termines

Mándame un mensaje con esto lleno:

```
GitHub usuario:        St4rVul ✓
Vercel dirección:      family-food-seven-lilac.vercel.app ✓
Neon:                  listo ✓ (us-east-2 Ohio, divine-water-82655296)
Google Client ID:      ________.apps.googleusercontent.com
Resend:                listo
Turnstile site key:    ________  (o "después")
```

Y ten a mano los 🔒 para pegarlos en `.env.local` cuando yo tenga el proyecto armado.

---

## Lista de correos de la familia

Para armar la lista blanca desde el arranque:

- [ ] danielsanmarquez84@gmail.com → **dueño**
- [ ] ________________________ → adulto
- [ ] ________________________ → menor
- [ ] ________________________ → menor

> **menor**: agrega cosas al carrito y ve recetas, pero no ve montos ni facturas.
> Cualquiera que intente entrar sin estar en esta lista te dispara el OTP a tu correo.

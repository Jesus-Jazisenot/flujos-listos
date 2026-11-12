# n8n desde cero, en español

Guía para quien nunca ha usado n8n. Al final tendrás n8n corriendo y sabrás importar cualquier plantilla de Flujos Listos.

## 1. Qué es n8n (en 30 segundos)

n8n es una herramienta para automatizar tareas: "cuando pase **esto** (disparador), haz **aquello** (pasos)". Cada paso es un **nodo**; el conjunto es un **flujo** (workflow). Un flujo se puede guardar como un archivo `.json`, y eso es lo que vendemos: archivos que importas y listo.

Palabras que verás:

- **Disparador (trigger):** lo que arranca el flujo (un horario, un formulario, un clic manual).
- **Credencial:** el permiso para que n8n entre a tu Gmail, tu hoja de Google, tu Telegram. Se crea una vez y se reutiliza.
- **Ejecución:** cada vez que el flujo corre. Los planes de pago de n8n Cloud cuentan ejecuciones.
- **Activo (Active):** el interruptor que hace que el flujo corra solo. Apagado, solo corre cuando tú lo pruebas.

## 2. Opción A (recomendada): instalar con Docker en tu computadora

Requisito: Docker Desktop (Windows/Mac) o Docker Engine (Linux). Comando oficial de la documentación de n8n (consultada el 12-nov-2026, <https://docs.n8n.io/hosting/installation/docker/>):

```powershell
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 `
  -e GENERIC_TIMEZONE="America/Mexico_City" -e TZ="America/Mexico_City" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n
```

(En bash/Linux/Mac cambia el acento grave `` ` `` por `\`.) Abre <http://localhost:5678>, crea tu cuenta de dueño y listo. Cambia la zona horaria por la tuya: de ella depende la hora a la que corren los flujos programados.

Para que quede corriendo siempre, cambia `-it --rm` por `-d --restart unless-stopped`. Los datos viven en el volumen `n8n_data`; no lo borres.

**Límite real de esta opción:** si apagas la computadora, los flujos programados no corren. Sirve para aprender y para flujos que tú disparas; para algo que deba correr todos los días a las 7:30 necesitas una máquina siempre encendida (ver sección 4).

Alternativa sin Docker: instalar con npm (`npm install n8n -g` y luego `n8n`), requiere Node.js. Documentación: <https://docs.n8n.io/hosting/installation/npm/>.

## 3. Importar una plantilla de Flujos Listos

1. En n8n: **Workflows → Create workflow** (o el menú de los tres puntos).
2. Menú **⋯ → Import from file…** y elige el `.json`.
3. Verás una nota amarilla (Sticky Note) con instrucciones y un nodo **Configuración**: ahí cambias tus datos (ID de hoja, correo, chat de Telegram...). Es el único lugar donde tienes que escribir valores.
4. Los nodos con un triángulo rojo necesitan credencial: ábrelos y elige la tuya (la guía de cada flujo dice cuál crear).
5. Pulsa **Execute workflow** para probar. Revisa la salida de cada nodo (clic en el nodo).
6. Cuando todo funcione, activa el interruptor **Active** arriba a la derecha.

## 4. Opciones gratuitas o casi gratuitas para tenerlo 24/7 (límites verificados el 12-nov-2026)

| Opción | Costo | Límites verificados | ¿Sirve para flujos programados? |
|---|---|---|---|
| **Tu computadora + Docker** | $0 | Solo corre si está encendida | Solo si la dejas prendida |
| **n8n Cloud, prueba gratis** | $0 al inicio | n8n.io/pricing muestra "Start free trial · No credit card required". Después: Starter 20 EUR/mes (facturado anual), 2,500 ejecuciones/mes; Pro 50 EUR/mes, 10,000 | Sí, pero de pago después de la prueba |
| **Oracle Cloud "Always Free"** | $0 | Docs oficiales: hasta 2 VMs AMD micro y equivalente a 2 OCPU y 12 GB de RAM en Ampere A1 (Arm) para tenencias Always Free. Pide registro y a veces no hay capacidad disponible en tu región | Sí; requiere saber de Linux y configurar HTTPS |
| **Render, plan Free** | $0 | Docs oficiales: el servicio gratis se "duerme" tras 15 min sin tráfico y tarda ~1 min en despertar; al dormirse pierde los archivos locales; Postgres gratis expira a los 30 días | **No**: un flujo programado no despierta a un servicio dormido |
| **VPS barato** | desde ~US$3-7/mes según blogs de terceros (no verificado) | Tú lo administras | Sí |

Fuentes: <https://n8n.io/pricing/>, <https://render.com/docs/free>, <https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm> (todas consultadas el 12-nov-2026; los planes cambian, revísalos antes de decidir).

**Licencia:** n8n Community Edition es gratuita bajo la *Sustainable Use License*: puedes usarla para los fines internos de tu negocio o de forma personal/no comercial, pero no revenderla ni ofrecerla como servicio a terceros. Detalle: <https://docs.n8n.io/sustainable-use-license/>. Las plantillas de este paquete son archivos de flujo propios y sí se pueden usar en tu n8n.

## 5. Crear credenciales (las que usan nuestras plantillas)

### Telegram (aviso gratis al celular)

1. En Telegram busca **@BotFather** y envía `/newbot`; sigue las preguntas. El nombre de usuario debe terminar en `bot`.
2. Copia el **token** que te da. En n8n: **Credentials → Add → Telegram API**, pega el token (Access Token). Guarda.
3. Abre tu bot en Telegram y mándale cualquier mensaje ("hola").
4. En el navegador abre `https://api.telegram.org/bot<TU_TOKEN>/getUpdates`. Busca `"chat":{"id":123456789` : ese número es tu **chat ID**. Cópialo en el nodo *Configuración* (`telegram_chat_id`).

### Google (Sheets, Gmail, Drive)

n8n self-hosted no puede usar el botón "Sign in with Google" gestionado (solo n8n Cloud); debes crear tu propia app de Google. Pasos de la documentación oficial (<https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/>):

1. Entra a Google Cloud Console y crea un proyecto.
2. **APIs y servicios → Biblioteca:** habilita **Google Sheets API**, **Gmail API** y **Google Drive API** (Sheets también requiere la de Drive).
3. **Pantalla de consentimiento OAuth:** tipo **External**; agrega tu correo como **usuario de prueba**.
4. **Credenciales → Crear credenciales → ID de cliente de OAuth → Aplicación web.** En "URI de redireccionamiento autorizados" pega la **OAuth Redirect URL** que te muestra n8n en la credencial (en local: `http://localhost:5678/rest/oauth2-credential/callback`; Google permite localhost).
5. Copia **Client ID** y **Client Secret** a la credencial de n8n (Google Sheets OAuth2 API, Gmail OAuth2 API, Google Drive OAuth2 API) y pulsa **Sign in with Google**.

**Error muy común:** si la app está en estado "Testing" y tipo "External", el permiso **caduca a los 7 días** (lo dice la doc de n8n). Solución: vuelve a conectar la credencial, o publica la app en Google Cloud.

### Gemini (solo el flujo 04, con IA)

1. Crea una llave en Google AI Studio (<https://aistudio.google.com/apikey>). Tiene nivel gratuito con límites que Google cambia; revísalos en <https://ai.google.dev/pricing>.
2. En n8n: **Credentials → Add → Header Auth**; Name: `x-goog-api-key`, Value: tu llave.

## 6. Solución de problemas general

- **El flujo no corre solo:** ¿está el interruptor Active encendido? ¿tu n8n está encendido a esa hora? ¿la zona horaria es la correcta? (Los flujos de Flujos Listos usan `America/Mexico_City`; cámbiala en *Workflow settings → Timezone*.)
- **Webhook no responde:** la URL de prueba (`/webhook-test/...`) solo vive mientras pulsas "Listen for test event"; la de producción (`/webhook/...`) solo funciona con el flujo Active.
- **"The caller does not have permission" / 403 en Google:** falta habilitar la API en tu proyecto, o tu correo no está como usuario de prueba.
- **No me aparecen las credenciales al importar:** normal; las plantillas no incluyen credenciales (por seguridad). Ábrelos y elige la tuya.

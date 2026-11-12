# 01 · Formulario web → Google Sheets → aviso

Archivo: `plantillas/01-formulario-a-sheets-y-aviso.json`

## Qué resuelve

Alguien llena el formulario de tu página (o de tu landing, o de un chatbot) y: se guarda una fila en tu hoja de Google, y te llega un aviso al celular (Telegram) o al correo. Se acabó el "se me perdió ese cliente en el correo". Filtra spam básico (campo trampa) y datos incompletos, y responde al formulario con un OK o un error claro.

## Requisitos

- n8n funcionando (ver `00-n8n-desde-cero.md`).
- Una hoja de Google con estos encabezados en la fila 1, en una pestaña llamada `Contactos`: `Fecha`, `Nombre`, `Correo`, `Teléfono`, `Mensaje`.
- Credencial **Google Sheets OAuth2** y credencial **Telegram** (o **Gmail** si prefieres correo).
- Para recibir formularios desde internet, tu n8n debe ser accesible por una URL pública (n8n Cloud, un VPS, o un túnel). En `localhost` solo recibes pruebas desde tu propia computadora.

## Pasos

1. Importa el JSON. 2. En el nodo **Configuración** pon: `hoja_id` (el texto largo de la URL de tu hoja entre `/d/` y `/edit`), `pestana`, `telegram_chat_id` y, si usarás correo, `correo_aviso`.
3. Abre **Guardar en Google Sheets** y **Avisar por Telegram** y elige tus credenciales.
4. Si prefieres correo: desactiva el nodo de Telegram, activa **Avisar por correo (desactivado)** (clic derecho → Activate) y elige tu credencial de Gmail. No dejes ambos si no quieres doble aviso.
5. Prueba con **Execute workflow**, luego en otra terminal:

```powershell
curl.exe -X POST http://localhost:5678/webhook-test/formulario-contacto -H "Content-Type: application/json" -d "{\"nombre\":\"Ana\",\"correo\":\"ana@correo.com\",\"mensaje\":\"Quiero una cotización\"}"
```

6. Activa el flujo. La URL de producción es `https://TU-DOMINIO/webhook/formulario-contacto`.

En tu formulario HTML, el envío puede ser un `fetch` con JSON o un `<form method="POST">` normal (n8n recibe ambos). Agrega un campo oculto `sitio_web` que los humanos no ven: si un bot lo llena, el flujo lo descarta.

## Cómo adaptarlo

- **Más campos:** agrega la columna en la hoja, el campo en el nodo *Validar y limpiar* y la línea en *Guardar en Google Sheets*.
- **Otro destino:** reemplaza el nodo de Sheets por Airtable, Notion o una base de datos; el resto no cambia.
- **Responder al cliente:** agrega un nodo Gmail después de *Guardar* que envíe un "recibimos tu mensaje" a `{{ $json.Correo }}`.
- **CORS:** si el formulario está en otro dominio y el navegador bloquea la respuesta, en el nodo Webhook → Options → *Allowed Origins (CORS)* pon tu dominio.

## Errores comunes

- Responde 400 "faltan datos": el JSON no trae `nombre`, `mensaje` y (`correo` o `telefono`).
- "Column names were updated after the node's setup": los encabezados de la hoja no coinciden (mayúsculas, acentos: `Teléfono`).
- El aviso no llega: ¿escribiste a tu bot al menos una vez? ¿el chat ID es el tuyo y no el del bot?
- Funciona en prueba pero no en producción: el flujo no está *Active* o usas la URL `-test`.

## Estado de pruebas (honesto)

- **Ejecutado de punta a punta en un n8n 2.40.7 real** (Docker), con el webhook activo y peticiones HTTP reales: formulario válido → 200; sin mensaje → 400; campo trampa → 400. La **escritura en Google Sheets y el envío a Telegram/Gmail se sustituyeron por nodos de prueba** que evalúan las mismas expresiones; esos nodos reales **solo se validaron por importación y por sus parámetros contra el paquete de nodos instalado**, no se ejecutaron contra cuentas reales.

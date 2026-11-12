# 05 · Respaldo semanal de una hoja de cálculo

Archivo: `plantillas/05-respaldo-semanal-hoja.json`

## Qué resuelve

Tu hoja de Google (clientes, inventario, cuentas) es el corazón de tu negocio y basta un borrado accidental para perder meses de trabajo. Cada lunes a las 6:00 este flujo hace una **copia completa** en tu Drive llamada `Respaldo - nombre - 2026-11-16` y te avisa. Si la copia falla, **te avisa del error**: no falla en silencio.

## Requisitos

- Credencial **Google Drive OAuth2** (ver guía 00; habilita *Google Drive API*).
- Credencial **Telegram** (o **Gmail** para el aviso).
- Espacio en tu Drive (cada copia ocupa lo que pese la hoja; las cuentas gratuitas tienen 15 GB compartidos).

## Pasos

1. Importa el JSON.
2. **Configuración:** `hoja_id` (de la URL de tu hoja), `nombre_hoja` (como quieres que se llame la copia), `telegram_chat_id` (o `correo_aviso`).
3. Elige credenciales en **Copiar la hoja en Drive** y en los nodos de aviso (hay dos: éxito y error).
4. Prueba con *Execute workflow*: debe aparecer `Respaldo - ...` en la carpeta de tu hoja y llegarte el aviso.
5. Activa el flujo.

## Cómo adaptarlo

- **Aviso por correo:** activa los nodos "(desactivado)" y desactiva los de Telegram.
- **Otra frecuencia:** cambia el disparador (diario, mensual).
- **Guardar en otra carpeta:** en el nodo de Drive apaga *Copy In The Same Folder* y elige carpeta destino.
- **Varias hojas:** duplica el flujo o usa un nodo *Loop Over Items* con una lista de IDs.
- **Exportar a Excel/CSV:** en lugar de copiar, usa Drive → *Download* con conversión; la copia nativa conserva fórmulas y formato, por eso es la predeterminada.

## Limitaciones honestas

- Es una copia **dentro de tu mismo Google Drive**: protege de borrados y errores, no de perder la cuenta de Google.
- El flujo nunca borra respaldos viejos (a propósito). Revisa la carpeta de vez en cuando.
- Si tu hoja usa scripts (Apps Script), comprueba tú mismo que la copia los conserve antes de confiar en ella; no lo verifiqué.

## Errores comunes

- "File not found": el `hoja_id` está mal o tu cuenta no tiene acceso al archivo.
- Llega el aviso de error con "insufficient permissions": falta la API de Drive en tu proyecto o la credencial caducó (7 días si la app está en *Testing*, ver guía 00).

## Estado de pruebas (honesto)

- **Lógica ejecutada en n8n 2.40.7 real**: ruta de éxito (arma nombre con fecha y aviso) y ruta de error (la salida de error del nodo de Drive dispara el aviso de falla). El nodo de Drive y Telegram **se simularon** (Drive reemplazado por un nodo de prueba; el error se forzó con una excepción). **El nodo real de Google Drive no se ejecutó contra Google**; solo se validó por importación y parámetros.

# Educa Inclusiva Panamá

## Documentación

## Levantar el sistema paso por paso

El proyecto usa el servidor Node.js incluido; no usa XAMPP. Necesitas Node.js 20 o posterior.

1. Instala Node.js desde [nodejs.org](https://nodejs.org/) si aún no está instalado.
2. Abre PowerShell en la carpeta del proyecto (la carpeta que contiene `package.json`).
3. Instala las dependencias, solo la primera vez:

   ```powershell
   npm install
   ```

4. Crea tu archivo local de configuración solo si aún no existe:

   ```powershell
   if (-not (Test-Path .env)) { Copy-Item .env.example .env }
   ```

5. Abre `.env` con VS Code y configura el correo:
   - La plantilla usa Gmail: `SMTP_PROVIDER=gmail`, `SMTP_USER=suzuvava@gmail.com` y `SMTP_FROM=suzuvava@gmail.com`.
   - En `SMTP_PASS`, agrega en tu archivo local la contraseña de aplicación de esa cuenta; no uses la contraseña normal de Gmail ni la compartas por el chat.
   - Si eliges Outlook/Microsoft 365, configura sus credenciales y asegúrate de que la cuenta tenga SMTP AUTH habilitado.
   - Deja `APP_BASE_URL=http://127.0.0.1:4173` cuando ejecutes el servidor en esta computadora, o configura `APP_URL` con la URL pública/local correcta para los enlaces de restablecimiento. Los correos de verificación se enviarán con remitente `suzuvava@gmail.com`.

   El registro y la verificación por correo requieren SMTP configurado. El remitente que se muestra en pantalla corresponde a `SMTP_FROM`. Si no hay SMTP configurado, la solicitud de restablecimiento imprime el enlace en la consola del servidor. No compartas ni subas el archivo `.env`.

6. Inicia el servidor:

   ```powershell
   npm start
   ```

   La consola debe mostrar `Servidor disponible en http://127.0.0.1:4173`.
7. Abre `http://127.0.0.1:4173` en el navegador; tanto `/` como `/index.html` cargan `educa_inclusiva_panam.html`. Usa **Iniciar sesión** o **Crear cuenta** desde el encabezado. Para registrarte, completa nombre, apellido, correo y contraseña, acepta los términos y condiciones e ingresa el código de 6 dígitos que llegue a tu correo. El código vence en 10 minutos, permite hasta 5 intentos y puede reenviarse cada 60 segundos; después inicia sesión.
8. Para recuperar una contraseña, elige **¿Olvidaste tu contraseña?** en el inicio de sesión, escribe el correo y abre el enlace que llegue por correo (o el que aparezca en la consola si no hay SMTP). El enlace vence en 30 minutos y solo se usa una vez. En la página de recuperación, escribe y confirma la nueva contraseña (8–128 caracteres) y vuelve al inicio de sesión. El formulario de login acepta tanto las contraseñas existentes como las restablecidas.
9. El botón **Subir recurso** siempre aparece en la biblioteca. Con una sesión iniciada y el correo verificado, abre el formulario; sin sesión, ofrece enlaces para iniciar sesión o registrarse. Para detener el servidor, vuelve a PowerShell y presiona `Ctrl+C`; para levantarlo en otra ocasión, repite `npm start`.

## Organización de archivos, recursos y cuentas

El proyecto separa los archivos por función: `src/server.js` contiene el servidor; `public/pages/` las páginas HTML; `public/scripts/` los JavaScript del navegador; `public/styles/` los estilos; `public/assets/` las imágenes y videos; y `data/` los JSON de datos. El servidor conserva las rutas públicas (`/`, `/auth`, `/app.js`, `/styles.css`, `/resources.json` y `/assets/...`) para que los enlaces existentes sigan funcionando.

Los recursos y sus adjuntos pueden consultarse sin iniciar sesión. El botón **Subir recurso** permanece visible; si no hay una sesión activa, muestra opciones para iniciar sesión o registrarse y no abre el formulario. Con una sesión verificada, el menú incluye un enlace al panel de administración, donde cada usuario solo puede gestionar sus propios recursos. Cada recurso admite hasta cinco archivos, de 200 MB cada uno.

Los metadatos públicos se guardan en `data/resources.json`. Las cuentas se guardan localmente en `data/users.json`, fuera de `public/`; el archivo y sus copias de respaldo están excluidos de Git y el servidor no los ofrece como archivos públicos. No los agregues manualmente al repositorio ni uses `git add -f`. `/api/me` solo devuelve los datos de la cuenta de la sesión verificada. No existe una ruta para listar cuentas; los perfiles públicos se consultan individualmente mediante `/api/profiles/<userId>` y nunca incluyen correo ni datos de autenticación.

En cada objeto de `attachments`, `size` es opcional y se expresa en bytes. Si omites la propiedad, `/resources.json` obtiene el tamaño real del archivo y lo incluye en la respuesta, sin modificar `data/resources.json`. También puedes escribir `"size": null` si prefieres dejar el tamaño sin especificar. Los archivos subidos desde el formulario guardan automáticamente su tamaño real.

Cada adjunto se ordena primero por propietario, después por categoría y finalmente por extensión: `uploads/resources/user_<UUID>/<categoría>/<extensión>/`. Las categorías son `documents`, `spreadsheets`, `presentations`, `images`, `videos` y `audio`; por ejemplo, un PDF queda en `uploads/resources/user_<UUID>/documents/pdf/`. Al iniciar el servidor, los archivos adjuntos existentes se migran a esta estructura y sus rutas se actualizan en los metadatos.

Al publicar el código en GitHub, sube solo los archivos de la aplicación, nunca `users.json`, sus respaldos ni `.env`. Si un archivo privado se sube por error, quitarlo del último commit no lo elimina del historial de Git: hay que avisar a los desarrolladores y tratar esos datos como expuestos.

Las sesiones activas se borran al reiniciar el servidor, pero las cuentas y los recursos permanecen en la computadora donde se ejecuta.

## Panel de recursos, portadas y consenso

### Perfiles de autor y edición

En el panel de administración, **Editar perfil** abre un formulario flotante que permite cambiar nombre, apellido, correo, ocupación, descripción y avatar. Para cambiar el correo, solicita un código que se envía al correo actual; configura `SUPPORT_EMAIL` para publicar el enlace de soporte (como alternativa se usa `SMTP_FROM` o `SMTP_USER`). Para cambiar la contraseña se solicita la actual, se confirma la nueva y se avisa que todas las sesiones, incluida la actual, se cerrarán. El perfil transmite únicamente nombre, apellido, ocupación, descripción e imagen; no distribuye el correo a otros visitantes.

Los avatares admiten PNG, JPEG, GIF o WebP de exactamente 500 × 500 píxeles y hasta 20 MB. Se guardan en `public/assets/profile-avatars/`; el archivo anterior se elimina al reemplazarlo o quitarlo. Si la red falla, los campos de texto se conservan como borrador en `localStorage` del dispositivo; los archivos de imagen deben seleccionarse de nuevo. El servidor usa WebSocket en `/ws` para avisar a los clientes conectados de cambios de perfil y recursos.

El enlace **Subido por…** abre el perfil público del autor, incluso sin iniciar sesión. Muestra sus publicaciones visibles, nivel, formato, área y fechas; los recursos anteriores a la captura de fechas muestran “Fecha no disponible”. Los visitantes sin sesión pueden consultar perfiles y previsualizar recursos, pero el servidor bloquea la descarga y apertura de archivos adjuntos de publicaciones de usuarios. Deben iniciar sesión para acceder a esos adjuntos. Los recursos no listados solo aparecen en el perfil cuando se visita con el enlace autorizado.

Con una sesión verificada, selecciona tu nombre en el encabezado para abrir el **Panel de Administración de Recursos** sin recargar la página. El panel consulta `/api/my/resources`, muestra una fila por recurso propio y permite subir, editar o eliminar recursos. La descripción se puede expandir para leerla completa.

Cada recurso puede tener una portada PNG o JPEG de **exactamente 500 × 500 píxeles** y hasta **10 MB**. El navegador comprueba tipo, dimensiones y peso antes de enviar el archivo; Node.js repite la validación y guarda las portadas en la carpeta privada del propietario. Los recursos anteriores reciben los campos `visibility`, `sharedWith` y `coverImage` al iniciar el servidor; las rutas existentes de sus adjuntos no se cambian. Mientras no haya portada, la interfaz usa `public/assets/resource-placeholder.svg`.

La visibilidad admite:

- **Público:** aparece en la biblioteca y puede consultarse sin iniciar sesión.
- **Enlistado:** no aparece en los listados generales; el panel genera un enlace directo y permite asignar hasta 20 cuentas verificadas mediante sus correos. El enlace funciona como un código compartible; cualquiera que lo reciba puede abrir el recurso.
- **Privado:** solo lo puede consultar su propietario.

La API aplica estas reglas también a la descarga de adjuntos y portadas; no depende de ocultar botones en el navegador. Los recursos compartidos con una cuenta aparecen en **Compartidos conmigo** al abrir el panel.

El **Blog de consenso** general y las conversaciones de cada recurso requieren una cuenta verificada para publicar. Las entradas y los comentarios admiten edición por su autor; el propietario del recurso también puede moderar y borrar conversaciones de ese recurso. Las respuestas conservan su estructura anidada. Los datos se guardan en `data/discussions.json`.

Los cambios dinámicos anuncian el estado de carga y guardado, errores de portada, cambios de visibilidad y acciones de conversación mediante regiones ARIA `live`. Los paneles y diálogos administran el foco y permiten cerrarse con Escape.

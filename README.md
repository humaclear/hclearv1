# HumaClear — sitio web y portal de clientes

Sitio estático (HTML + módulos ES + CSS, **sin build y sin npm**) con dos partes:

1. **Sitio comercial** (`index.html`): servicios con demos, paquetes, proceso, IA, video, nosotros y contacto.
2. **Portal de clientes** (`portal.html`) y **panel de administración** (`admin.html`), con Firebase Authentication + Firestore.

> **Al actualizar archivos, conserva tu `js/config.js`.** Es el único archivo con tus datos de Firebase. Si subes una versión nueva del proyecto, no lo reemplaces por el que viene vacío.

---

## Estructura (todo va en la raíz del repositorio)

```
index.html  portal.html  admin.html  privacidad.html  diagnostico.html
vercel.json  firestore.rules  README.md
assets/  favicon.svg  humaclear-poster.jpg  (humaclear.mp4  humaclear.webm — opcionales)
css/     styles.css (sitio + tokens)   app.css (portal y admin)
js/      config.js  catalog.js  chart.js  site.js  portal.js  admin.js  auth-ui.js
         backend.js  backend-firebase.js  backend-demo.js
```

- `js/config.js`: tu `firebaseConfig`, datos de contacto y el modo (demo o Firebase).
- `js/catalog.js`: servicios, paquetes, proceso, íconos y textos. **Edita aquí precios y descripciones.**
- `js/backend.js`: interfaz única; usa `backend-firebase.js` si hay `apiKey`, o `backend-demo.js` si está vacío.
- `diagnostico.html`: revisa que todos los archivos estén publicados y que `config.js` sea válido. Ábrelo en `https://tu-dominio/diagnostico` si algo no carga.

## Probar en tu computadora

Los módulos ES no funcionan abriendo el archivo con doble clic; necesitas un servidor local. Desde la carpeta del proyecto:

```bash
python3 -m http.server 8000
```

Luego abre `http://localhost:8000`. Cualquier servidor estático sirve (por ejemplo, la extensión Live Server de VS Code).

### Modo demo

Si `apiKey` en `js/config.js` está vacío, el portal funciona **sin Firebase**, con datos guardados solo en tu navegador:

| Cuenta | Contraseña | Qué ves |
|---|---|---|
| `admin@demo.mx` | `demo1234` | Panel de administración |
| `cliente@demo.mx` | `demo1234` | Cliente con 2 servicios, documentos y una solicitud |
| `nuevo@demo.mx` | `demo1234` | Cliente recién creado, sin servicios |

La pantalla de inicio de sesión muestra botones de acceso rápido. Para reiniciar los datos de la demo, borra los datos del sitio en tu navegador.

---

## Configurar Firebase (paso a paso)

1. Entra a https://console.firebase.google.com y crea un proyecto (puedes desactivar Google Analytics).
2. En la pantalla del proyecto, agrega una **app web** (ícono `</>`). Copia el objeto `firebaseConfig` y pégalo en `js/config.js`, respetando **comillas y comas**:
   ```js
   export const firebaseConfig = {
     apiKey: "AIza…",
     authDomain: "tu-proyecto.firebaseapp.com",
     projectId: "tu-proyecto",
     storageBucket: "tu-proyecto.appspot.com",
     messagingSenderId: "123…",
     appId: "1:123…:web:abc…",
   };
   ```
3. **Authentication → Método de acceso → Correo electrónico/contraseña → Habilitar.** (En la consola nueva, Authentication puede estar dentro del menú "Seguridad".)
4. **Authentication → Configuración → Dominios autorizados**: agrega tu dominio de Vercel (por ejemplo `humaclear-mx.vercel.app`) y tu dominio propio si tienes uno.
5. **Firestore Database → Crear base de datos**: modo **producción**, región cercana (por ejemplo `us-central1` o `nam5`).
6. **Firestore → Reglas**: pega el contenido completo de `firestore.rules` y presiona **Publicar**.
7. **Crea el primer administrador** (solo una vez, a mano):
   1. Authentication → Usuarios → **Agregar usuario** (correo y contraseña).
   2. Copia el **UID** de ese usuario.
   3. Firestore → **Iniciar colección** `users` → ID del documento = **ese UID**, con estos campos:

      | Campo | Tipo | Valor |
      |---|---|---|
      | `name` | string | Tu nombre |
      | `company` | string | HumaClear |
      | `email` | string | tu correo |
      | `role` | string | `admin` |
      | `active` | boolean | `true` |
      | `createdAt` | string | `2026-10-12T00:00:00.000Z` (fecha en texto ISO) |
8. Entra a `/admin` con esa cuenta y crea a tus clientes desde **Nuevo cliente**. El panel crea la cuenta sin cerrar tu sesión (usa una segunda instancia de Firebase).
9. Sube el proyecto a GitHub (**los archivos en la raíz del repositorio**, no dentro de otra carpeta) y conéctalo con Vercel: *Framework Preset: Other*, sin comando de build, directorio raíz.

### Cómo se protegen los datos

- No hay registro abierto: solo el administrador crea cuentas.
- Un cliente **solo lee lo suyo** y solo si `active == true`. Si desactivas a un cliente en el panel, pierde el acceso al instante.
- Un cliente solo puede **crear** solicitudes propias; no puede editar contratos, documentos ni perfiles.
- Nadie puede darse permisos de administrador: solo un admin escribe en `users`.
- Cualquiera puede enviar una cotización (`leads`), con validación estricta; solo el admin las lee.
- La `apiKey` de Firebase es pública por diseño; la seguridad está en `firestore.rules`.

### Lista de pruebas con Firebase real

- [ ] Entras a `/admin` con el administrador.
- [ ] Creas un cliente y **sigues con tu sesión de admin**.
- [ ] El cliente entra a `/portal` y ve solo sus servicios y documentos.
- [ ] Con dos clientes, uno **no** ve los datos del otro.
- [ ] Al desactivar a un cliente, ve "Tu acceso está en pausa".
- [ ] Una cotización enviada desde el sitio aparece en **Prospectos**.

---

## Panel de administración

- **Clientes**: lista, interruptor de acceso, botón **Nuevo cliente**.
- **Detalle de cliente**: un editor por servicio. Marca *Contratado*, elige estado, URL del sistema, nota, hasta 4 indicadores y la gráfica. Los datos van **una línea por dato** con el formato `etiqueta: valor`:
  ```
  Compensación: 31
  Liderazgo: 24
  Crecimiento: 19
  ```
  Si una línea está mal escrita, el panel te dice cuál. También puedes agregar **documentos** (enlaces de Drive, Dropbox, etc.) y usar **Ver como cliente** para revisar lo que verá.
- **Solicitudes**: responde y cambia el estado (abierta, en proceso, resuelta). El cliente ve tu respuesta en su portal.
- **Prospectos**: cotizaciones del sitio, con estado y opción de eliminar.

## Agregar el video

1. Exporta tu video en **MP4 (H.264)** y, si puedes, también en **WebM**.
2. Guárdalos como `assets/humaclear.mp4` y `assets/humaclear.webm`.
3. (Opcional) Reemplaza `assets/humaclear-poster.jpg` con una imagen de portada de 1280×720.

La sección "Míralo en acción." aparece sola cuando el video carga. Si los archivos no existen, la sección queda oculta y no deja un hueco.

## Si algo no carga

1. Abre `/diagnostico`: marca **FALTA** en los archivos que no se subieron.
2. Revisa que `js/config.js` tenga comillas y comas correctas (el diagnóstico te dice si tiene un error).
3. El sitio público **nunca depende de Firebase**: si el portal falla, el sitio sigue funcionando y el formulario de cotización ofrece enviar los datos por correo.
4. Si Firebase no responde en 12 segundos, el portal muestra un mensaje con **Reintentar** en lugar de quedarse cargando.

## Antes de publicar

- Completa el domicilio en `privacidad.html` (busca `[completar antes de publicar]`).
- Revisa precios y textos en `js/catalog.js`.

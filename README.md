# SyC Óptica — Sitio web con panel de administración

Sitio estático moderno (no necesita servidor propio ni base de datos) con un panel
que permite editar **todo** el contenido público sin tocar código.

## Estructura

```
index.html            → sitio público
admin.html            → panel de administración
css/site.css          → estilos del sitio
css/admin.css         → estilos del panel
js/content-default.js → contenido "de fábrica" (textos, promos, marcas, etc.)
js/store.js           → capa de datos compartida (guardado, respaldos, imágenes)
js/site.js            → arma el sitio a partir del contenido
js/admin.js           → lógica del panel (login, edición, respaldos)
assets/               → imágenes fijas (logo)
```

## Cómo publicarlo

Subí la carpeta completa a cualquier hosting estático: **Netlify, Vercel,
GitHub Pages, Cloudflare Pages** o el hosting que ya tengas (por FTP).

Para que el panel responda en la ruta `/admin` (además de `/admin.html`):

- **Netlify**: creá un archivo `_redirects` con la línea `/admin /admin.html 200`
- **Vercel**: en `vercel.json` → `{"rewrites":[{"source":"/admin","destination":"/admin.html"}]}`
- **Apache (hosting clásico)**: en `.htaccess` → `RewriteRule ^admin$ admin.html [L]`

## Panel de administración

1. Entrá a `tudominio.com/admin.html`
2. Acceso inicial de prueba: usuario **admin** / contraseña **admin123**
3. **Cambiá la contraseña apenas entres** en *Ajustes → Seguridad* (mínimo 8 caracteres).

Desde el panel podés administrar: portada (textos, botones, banner, colores),
promociones (con fechas de inicio/fin y destacadas), especialidades (carrusel),
marcas (cinta deslizante), galería (con compresión automática de imágenes),
publicaciones tipo blog, la sección de Instagram y las redes sociales del pie.

Los cambios se aplican al tocar **Guardar cambios** y se ven al instante en el sitio.

### Dónde se guarda el contenido (importante)

Al ser un sitio sin servidor, el contenido editado se guarda **en el navegador
de la computadora donde se edita** (localStorage). Eso implica:

- Los cambios guardados desde el panel se ven al instante **en esa misma
  computadora/navegador** (ideal para armar y previsualizar todo). Los
  visitantes, en cambio, ven el contenido de `js/content-default.js` que está
  subido al hosting. Para publicar tus cambios para todo el mundo el flujo es:
  1. Editá todo desde el panel y guardá.
  2. En *Ajustes → Respaldos* tocá **Exportar respaldo** (archivo JSON).
  3. Abrí `js/content-default.js`, reemplazá el objeto después de
     `window.SYC_DEFAULT =` por el contenido del JSON exportado, y volvé a
     subir ese único archivo al hosting. Listo: todos ven la nueva versión.
     (El JSON exportado también sirve como respaldo o para pasar el contenido
     a otra computadora con *Importar respaldo*.)
- Alternativa profesional: cuando quieras que el panel publique directo para
  todos los visitantes sin ese paso, hay que sumar un backend (por ejemplo
  Supabase o Firebase, gratis en su plan inicial). La estructura del código ya
  está preparada: solo habría que reemplazar `js/store.js` por una versión que
  lea/escriba en esa base de datos.

### Seguridad — qué cubre y qué no

- La contraseña se guarda **cifrada con SHA-256**, nunca en texto plano.
- La sesión **se cierra sola por inactividad** (configurable en Ajustes).
- Todo el contenido se **escapa** antes de mostrarse (protección XSS) y no hay
  base de datos SQL ni formularios que envíen datos, por lo que la inyección
  SQL y el CSRF no aplican a esta arquitectura.
- Límite honesto: al no haber servidor, el login protege el *panel de edición
  de tu navegador*, pero no es una barrera de servidor real. Nadie puede
  modificar el sitio que ven los demás desde afuera (el hosting es de solo
  lectura), que es lo que importa. Si más adelante se suma un backend, ahí sí
  corresponde autenticación del lado del servidor.

## Instagram

- El botón "Seguinos", el ícono del menú y el del pie ya apuntan a
  `instagram.com/sycoptica` (editable en el panel).
- **Feed curado (funciona ya, sin configurar nada):** en *Panel → Instagram*
  pegá los enlaces de las publicaciones que quieras mostrar (Compartir →
  Copiar enlace). Se incrustan con el reproductor oficial de Instagram y al
  tocar una se abre la publicación real.
- **Feed 100% automático (opcional):** Instagram solo permite leer las últimas
  publicaciones mediante su API oficial con un *token de acceso*. Se obtiene
  gratis creando una app en `developers.facebook.com` (producto **Instagram
  API**) y vinculando la cuenta @sycoptica. Pegá el token en *Panel →
  Instagram → Actualización automática* y el sitio mostrará solo las últimas
  6 publicaciones sin tocar nada más. Los tokens de larga duración vencen a
  los 60 días y conviene renovarlos.

## Rendimiento y SEO

- Imágenes con `loading="lazy"` y compresión automática al subirlas (máx.
  1400 px, JPEG 82 %).
- Sin frameworks ni librerías: ~30 KB de JavaScript propio.
- Meta tags, Open Graph y datos estructurados (schema.org `Optician`) para
  Google Maps/Búsqueda incluidos en `index.html`.
- Animaciones que respetan `prefers-reduced-motion` y navegación accesible
  por teclado.

## Ampliaciones futuras

El código está separado por responsabilidad (datos / render / panel), en
español y comentado. Para agregar una sección nueva: sumá la clave al objeto
de `content-default.js`, una función de render en `site.js` y una pantalla en
`admin.js` siguiendo cualquiera de las existentes como plantilla.

# conservasramonpena.com

Copia estática de la web de Conservas Ramón Peña (antes en WordPress + Divi), publicada con GitHub Pages en https://conservasramonpena.com.

## Estructura

- `site/`: la web publicada (HTML, CSS, JS e imágenes). Incluye `CNAME`, `robots.txt`, `sitemap.xml` y `404.html`.
- `tools/mirror.py`: regenera `site/` rastreando la WordPress original.
- `tools/serve.py`: servidor local para previsualizar (admite Range, necesario para los vídeos MP4).
- `.github/workflows/pages.yml`: publica `site/` en GitHub Pages en cada push a `main`.

## Uso

```sh
python tools/serve.py 8000          # previsualizar en http://127.0.0.1:8000/
python tools/mirror.py site         # regenerar desde la WordPress (si sigue en línea)
python tools/mirror.py site --post  # solo el postproceso (extras, formularios, rutas)
```

Las rutas son relativas a cada página (`../wp-content/...`; el postproceso de `mirror.py` las convierte),
así que la web funciona tanto en la raíz del dominio como en https://eloydelgado.github.io/conservasramonpena.com/.
La `404.html` calcula su `<base>` según dónde se sirva.

## DNS para GitHub Pages

El dominio principal es el apex (`conservasramonpena.com`, sin `www`), como en la WordPress.

- `conservasramonpena.com` (apex) → registros A: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- `www` → registro CNAME a `eloydelgado.github.io` (GitHub redirige `www` al apex)

Ahora el dominio pasa por Cloudflare. Si se mantiene, los registros deben ir en modo «solo DNS» (nube gris)
al menos hasta que GitHub emita el certificado HTTPS.

## Cloudflare

La WordPress pasaba por Cloudflare, que ofusca los correos (`/cdn-cgi/l/email-protection`) y añade sus scripts.
Fuera de Cloudflare eso no existe, así que el postproceso de `mirror.py` decodifica los correos a texto y
`mailto:` normales y quita los scripts de `/cdn-cgi/`.

## Prefijo /2023/

La web se montó primero en `/2023/` y quedaban enlaces con ese prefijo: los enlaces de las columnas del
catálogo, el pie (política de privacidad) y las imágenes de fondo de las fichas. En WordPress `/2023/<página>/`
redirigía a `/<página>/`, y `/2023/wp-content/...` daba 404, así que esas imágenes no se veían. `mirror.py`
quita el prefijo al rastrear y en el postproceso, de modo que la copia estática enlaza directamente a las
páginas y muestra las imágenes.

## Formularios

Los formularios se envían con [extras/assets/js/formularios.js](extras/assets/js/formularios.js): valida en el
navegador y llama a la Cloud Function `contactoWeb` del proyecto Firebase `grupodelgado-prod` (la misma que usan
delgadoseleccion.com, cepasconhistoria.com y conservaslabrujula.com), protegida con App Check + reCAPTCHA Enterprise.

- **Hazte distribuidor** (`/hazte-distribuidor/`): formulario de contacto de Divi (nombre, correo, teléfono,
  empresa, localidad y mensaje, todos obligatorios). Es el único formulario de la web.
- Los ficheros de `extras/` se copian a `site/`, y el `<script>` se añade a las páginas con formulario
  mediante `python tools/mirror.py site --post` (también se hace al regenerar).
- La clave de reCAPTCHA Enterprise debe tener `conservasramonpena.com` en su lista de dominios permitidos.
- Si se deja vacía `recaptchaSiteKey`, el formulario abre el correo del visitante dirigido a web@grupo-delgado.com.

## Limitaciones

- El buscador de Divi (`?s=`) necesita PHP y no funciona en estático.
- La imagen `LP-800452DOP-MEJILLONES-PICANTES-AJO-Y-GUINDILLA-OL120-16_20-bodegon.png` ya faltaba en la WordPress
  (la referencia una hoja de estilos de Divi).

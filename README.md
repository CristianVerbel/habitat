# KAI — Landing Page

Landing page de una sola página para el proyecto inmobiliario KAI (Barranquilla). Sitio **100% estático** — HTML + CSS + JS puro, sin build step, sin dependencias de servidor ni base de datos.

## Estructura

```
kai-landing/
├── index.html          ← página completa (todo el contenido y estilos)
├── assets/              ← copia local de respaldo; index.html usa las de Supabase Storage (ver abajo)
│   ├── kai-wordmark.jpg     ← logotipo cinemático (hero)
│   ├── og-image.jpg         ← imagen para previews de redes sociales
│   ├── favicon-32.png
│   ├── favicon-192.png
│   └── apple-touch-icon.png
├── robots.txt
└── README.md            ← este archivo
```

## Imágenes: servidas desde el Supabase de Vendu

Las imágenes (`og:image`, favicons, wordmark) están subidas al bucket público
`kai-landing` en el proyecto Supabase de Vendu y `index.html` las referencia
directamente desde ahí:
`https://lahexnphsecabgdtpska.supabase.co/storage/v1/object/public/kai-landing/assets/...`

**La página (`index.html`) en sí NO puede vivir en ese mismo Supabase.** Se intentó
servirla tanto vía Storage como vía Edge Function, y en ambos casos la plataforma
fuerza `Content-Type: text/plain` + `Content-Security-Policy: sandbox` sobre
cualquier contenido que detecta como HTML — es una protección anti-XSS/phishing
a nivel de plataforma (no hay opción de proyecto para desactivarla), así que el
JS, el CSS y el formulario simplemente no se ejecutarían. Por eso `index.html`
debe desplegarse en un host estático normal (ver "Desplegar" abajo); todo lo
demás — captura de leads, CRM, campaña, imágenes — sí vive 100% en el Supabase
de Vendu.

## ⚠️ Antes de publicar — 2 cosas que EDITAR

**1. Número de WhatsApp.** Abre `index.html`, busca (al final del archivo, dentro de `<script>`):
```js
const WHATSAPP_NUMBER = "573000000000"; // reemplazar por el número real
```
Reemplázalo por el número real en formato internacional, solo dígitos (código país + número, sin `+`, sin espacios ni guiones). Ejemplo Colombia: `573001234567`.

**2. Correo de contacto.** Busca `[correo]` en `index.html` (sección de contacto, `id="contacto"`) y reemplázalo por el correo real.

Opcionalmente también:
- El mapa de Google (`id="ubicacion"`) ya apunta a la dirección real (Cra 46 # 70-131, Barranquilla) — no requiere API key ni configuración.
- El formulario de contacto, al enviarse, hace dos cosas en paralelo: (1) registra el lead en el **CRM B2B** (repo `vendusalestech`, campaña `kai-barranquilla`) vía la Edge Function pública `CRM_LEAD_ENDPOINT` (buscar en `index.html`), y (2) arma un mensaje con los datos capturados y abre WhatsApp directamente. La captura al CRM es silenciosa (no bloquea ni depende de que WhatsApp se abra); si el endpoint falla, el formulario igual funciona.

## Gestión de los leads (CRM)

Cada envío del formulario crea/actualiza en el CRM B2B de `vendusalestech`:
- Una **cuenta** (segmento `Inversionista KAI`, ciudad Barranquilla), sin duplicar en reenvíos del mismo teléfono.
- Un **negocio** en el pipeline (`prospecto`), con el valor estimado según la capacidad de inversión indicada.
- Una **tarea pendiente** (vence en 24h) para que un asesor haga seguimiento — aparece en la pestaña **Tareas** del CRM.

Para verlos: entrar al CRM B2B (`/admin/crm-b2b` en Vendu) y filtrar por segmento **Inversionista KAI** o ciudad **Barranquilla**. Detalle técnico del endpoint en `vendusalestech/docs/CRM_B2B.md` (Fase 9).

## Desplegar

Es un sitio estático puro — cualquiera de estas opciones funciona sin configuración adicional:

### Opción A — Vercel (recomendado, más simple)
```bash
cd kai-landing
npx vercel --prod
```
Sigue las instrucciones en pantalla (crear cuenta/login si es la primera vez). Vercel detecta automáticamente que es un sitio estático.

### Opción B — Netlify
```bash
cd kai-landing
npx netlify-cli deploy --prod --dir .
```

### Opción C — GitHub Pages
```bash
cd kai-landing
git init
git add .
git commit -m "KAI landing page"
git branch -M main
git remote add origin <URL_DEL_REPOSITORIO>
git push -u origin main
```
Luego activar GitHub Pages en la configuración del repositorio (Settings → Pages → Deploy from branch → main → /root).

## Probar localmente antes de publicar

```bash
cd kai-landing
npx serve .
```
Abre la URL que muestre en terminal (usualmente `http://localhost:3000`).

## Dominio propio

Una vez desplegado en Vercel o Netlify, ambas plataformas permiten conectar un dominio propio (ej. `kai.com.co`) desde su panel — Domains → Add Domain — y siguiendo las instrucciones de DNS que muestran (usualmente apuntar un registro CNAME o A).

## Notas de contenido

- Todas las cifras de rentabilidad en la página incluyen su nota legal correspondiente (marcada con *). **No eliminar esa nota** al editar textos.
- La página está sincronizada con el modelo financiero oficial vigente (escalera de precios $330M/$350M/$370M, renta desde 9% E.A. desde el segundo año). Si el modelo cambia, esta página debe actualizarse antes de seguir circulando.
- Nota interna: según la bitácora del proyecto, la publicación pública (pauta, redes) de material comercial debería esperar la confirmación de uso de suelo por curaduría. Este archivo puede desplegarse en cualquier momento para revisión privada; evaluar el momento de hacerlo público/indexable.

## Stack técnico

- HTML5 + CSS3 (variables nativas, grid, sin frameworks)
- JavaScript vainilla (sin dependencias, sin bundler)
- Tipografías: Google Fonts (Cormorant Garamond, Playfair Display, Inter) vía CDN
- Mapa: Google Maps embebido (iframe público, sin API key)
- 100% responsive (probado desde 390px hasta 1400px+)
- Sin cookies, sin analytics preinstalado (agregar Google Analytics / Meta Pixel si se requiere, antes de `</head>`)

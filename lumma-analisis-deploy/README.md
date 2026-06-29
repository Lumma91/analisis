# Lumma · Landing /analisis

Landing page de adquisición para Lumma con captura de leads vía HubSpot y tracking de conversión vía Meta Pixel.

## Estructura del proyecto

```
lumma-analisis-deploy/
├── index.html          ← Landing principal (también responde en /analisis)
├── gracias.html        ← Página de confirmación post-formulario
├── vercel.json         ← Configuración de Vercel (clean URLs, headers)
├── .gitignore
├── assets/             ← Aquí va la foto del equipo
└── README.md
```

## Pasos para desplegar en Vercel

### 1 · Subir foto del equipo

Antes de deployar, agregar la foto en `/assets/lumma-equipo.jpg` y descomentar el `<img>` en `index.html` (sección "Sobre Lumma" — línea con comentario `<!-- Cuando subas la foto... -->`).

**Recomendación de foto:** la del estudio limpio con stand de cámara móvil + persona modelando con periódico (la última de las 5 que compartiste). Funciona perfecto en crop circular y captura el "content day" que es central al método Lumma.

Si prefieres otra opción de las que compartiste:
- **Opción A** (recomendada): estudio + cámara móvil + modelo con periódico → más editorial
- **Opción B**: las dos personas grabando con equipo profesional (back view) → energía de equipo
- **Opción C**: presentadora con luz LED siendo filmada → muestra trabajo en evento real
- **Opción D**: gimbal recording en evento → dinámica pero específica de un cliente

### 2 · Subir a GitHub

```bash
cd lumma-analisis-deploy
git init
git add .
git commit -m "Initial commit: landing /analisis con Pixel + HubSpot"
git branch -M main
git remote add origin https://github.com/[tu-usuario]/lumma-analisis.git
git push -u origin main
```

### 3 · Desplegar en Vercel

1. Entra a [vercel.com](https://vercel.com), click "Add New" → "Project"
2. Importa el repo de GitHub que acabas de crear
3. Vercel detecta automáticamente como static site → no necesitas configurar nada
4. Click "Deploy"
5. En ~30 segundos tendrás el preview URL: `lumma-analisis-[xxx].vercel.app`

### 4 · Conectar dominio (opciones)

**Opción A · Subdominio dedicado** (recomendado para empezar):
- En Vercel project: Settings → Domains → Add `analisis.lummacreative.com`
- En tu proveedor DNS de lummacreative.com: agregar CNAME `analisis` → `cname.vercel-dns.com`
- URL final: `analisis.lummacreative.com` y `analisis.lummacreative.com/gracias`

**Opción B · Path en sitio principal** (requiere que lummacreative.com también esté en Vercel):
- Agregar `index.html` y `gracias.html` al proyecto principal
- Configurar rewrite en el `vercel.json` del proyecto principal
- URL final: `lummacreative.com/analisis` y `lummacreative.com/gracias`

## Integraciones configuradas

### Meta Pixel
- ID: `1001159735966041`
- Eventos disparados:
  - `PageView` al cargar cualquier página
  - `Lead` al enviar el formulario (con `eventID` para deduplicación)
  - `Lead` al cargar `/gracias` (backup, usa el mismo `eventID` para evitar duplicados)

### HubSpot
- Portal ID: `246101311`
- Form ID: `da48d492-db35-4892-91af-9cec496313b6`
- El formulario visible es el de Lumma (custom design). Los datos se envían en background a HubSpot via su Forms API.
- Campos enviados:
  - `firstname` ← Nombre
  - `email` ← Email
  - `phone` ← WhatsApp
  - `message` ← String concatenado con tipo de negocio + presupuesto + fuente

**Nota:** si en HubSpot tienes propiedades personalizadas para "tipo de negocio" y "presupuesto mensual", podemos ajustar el JS para mapearlas directo. Hoy van consolidadas en el campo `message` para evitar dependencia de tu schema interno.

### Calendar (Google Calendar)
- Link: `https://calendar.app.google/SwFGAYMnVEpbNABv8`
- Aparece en `/gracias` como botón principal "Agenda directo →"

### WhatsApp
- Número: `+1 786 429 5504`
- Aparece en `/gracias` en la fila social

## Verificación post-deploy

Después de deployar, hacer estas pruebas:

1. **Visual mobile + desktop:** abrir en navegador, redimensionar de 320px → 1920px. Todo debe verse bien en cada ancho.
2. **Formulario:** enviar un test con tu propio email. Verificar:
   - Llega un lead a tu cuenta de HubSpot con los datos correctos
   - Redirige a `/gracias` exitosamente
   - Ves el evento Lead en Meta Events Manager (puede tardar 5-10 min)
3. **Calendar:** click en "Agenda directo →" — debe abrir el Google Calendar booking en nueva pestaña
4. **WhatsApp:** click en el link de WhatsApp — debe abrir chat con +1 786 429 5504

## Optimización aplicada

### Mobile-first
- Mobile: 1 columna, hero apilado, padding reducido, tipografía clamp() que escala con viewport
- Desktop ≥960px: hero 2 columnas, método 3 columnas, todo respira

### Performance
- Sin librerías JS externas (solo Google Fonts via preconnect)
- CSS inline (1 round trip)
- Animaciones puro CSS (sin GSAP/Framer)
- Smooth scroll nativo, IntersectionObserver nativo
- Lighthouse score esperado: 90+ en performance, 100 en accessibility/best practices/SEO

### Accesibilidad
- Labels asociados a inputs (id/for)
- aria-labels en links de logo
- `aria-hidden="true"` en decorativos
- Focus states visibles en form inputs
- Color contrast cumple WCAG AA

## Personalización futura

### Cambiar precios del FAQ
Buscar "Trabajamos en tres frentes" en `index.html` y editar los `<li>`.

### Cambiar copy del hero
Buscar el `<h1>` con "Que lo que haces" — la palabra "brille" usa `<em>` para el italic serif magenta.

### Cambiar colores
Editar las CSS variables en `:root` (líneas 18-30 del `<style>` de `index.html`):
- `--lime: #C8F03B`
- `--magenta: #E91560`
- `--cream: #F4EDE0`

### Mapear campos HubSpot personalizados
Editar la función `handleSubmit` en `index.html` (busca `hsPayload`) y agregar:

```js
{ objectTypeId: '0-1', name: 'tipo_de_negocio', value: data.tipo },
{ objectTypeId: '0-1', name: 'presupuesto_mensual', value: data.presupuesto }
```

(asumiendo que `tipo_de_negocio` y `presupuesto_mensual` son los internal names de tus propiedades en HubSpot).

---

**Stack:** HTML + CSS + JS vanilla · 0 dependencias · Static deploy en Vercel
**Tracking:** Meta Pixel + HubSpot Forms API
**Versión:** v3 final · Junio 2026

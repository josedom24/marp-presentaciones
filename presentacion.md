---
marp: true
theme: profesional
paginate: true
header: 'Marp — Presentaciones con Markdown'
footer: 'José Domingo · josedom24@gmail.com · 2026'
---

<!-- _class: portada -->
<!-- _paginate: false -->

# Presentaciones con **Marp**

## Markdown Presentation Ecosystem

<p>José Domingo &nbsp;·&nbsp; Abril 2026 &nbsp;·&nbsp; v2.0</p>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Introducción a Marp

## Qué es, para qué sirve y cómo funciona

---

## ¿Qué es Marp?

**Marp** convierte archivos Markdown en presentaciones HTML, PDF o PowerPoint usando un motor basado en Chromium.

> Escribe en Markdown, presenta como un profesional — sin tocar PowerPoint.

<div class="cols-2" style="margin-top:1rem">

<div>

### Por qué usarlo

- Sintaxis **simple y conocida**
- Control de versiones con **Git**
- Exportación a **HTML, PDF, PPTX**
- Temas completamente **personalizables**
- Integración con **VS Code** y CI/CD

</div>

<div>

### Por qué no PowerPoint

- Sin archivos binarios difíciles de versionar
- Sin dependencias de licencias
- Flujo **código → presentación** automatizable
- Reutilización de componentes y estilos
- **100% reproducible** en cualquier máquina

</div>

</div>

---

## Estructura de un archivo Marp

<div class="cols-60-40">

<div>

```markdown
---
marp: true          # Activa el motor Marp
theme: default      # Tema CSS
paginate: true      # Numeración de páginas
header: 'Título'    # Cabecera global
footer: 'Autor'     # Pie global
size: 16:9          # Relación de aspecto
---

# Primera diapositiva

Contenido en Markdown.

---

# Segunda diapositiva

Separadas por tres guiones ( --- ).
```

</div>

<div>

<div class="alerta alerta-info">
<span>ℹ️</span> <div>El bloque entre <code>---</code> inicial es el <strong>frontmatter</strong>. Configura toda la presentación.</div>
</div>

<div class="alerta alerta-ok" style="margin-top:0.8rem">
<span>✅</span> <div>Cada <code>---</code> en el cuerpo separa una diapositiva nueva.</div>
</div>

<div class="alerta alerta-warning" style="margin-top:0.8rem">
<span>⚠️</span> <div>Las directivas locales <code>&lt;!-- _clave: valor --&gt;</code> sobreescriben las globales solo en esa diapositiva.</div>
</div>

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Tipografía y Contenido

## Texto, listas, código y citas

---

## Formateo de texto

<div class="cols-2">

<div>

### Énfasis y estilo

| Sintaxis | Resultado |
|----------|-----------|
| `**texto**` | **negrita** |
| `*texto*` | *cursiva* |
| `~~texto~~` | ~~tachado~~ |
| `` `código` `` | `código` |

### Listas anidadas

- Elemento nivel 1
  - Sub-elemento A
  - Sub-elemento B
    - Nivel profundo
- Otro elemento

</div>

<div>

### Listas numeradas

1. Primer paso crítico
2. Segundo paso importante
3. Tercer paso opcional
   1. Sub-paso A
   2. Sub-paso B

### Combinado

- **Fase 1**: Diseño `v1.0`
- **Fase 2**: Implementación `v2.0`
- **Fase 3**: ~~Cancelado~~ → *Pospuesto*

</div>

</div>

---

## Bloques de código

```python
# Clase principal de la aplicación
class APIService:
    """Servicio REST con autenticación JWT."""

    def __init__(self, base_url: str, token: str):
        self.base_url = base_url
        self.headers  = {"Authorization": f"Bearer {token}"}

    async def get(self, endpoint: str) -> dict:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/{endpoint}",
                headers=self.headers,
                timeout=30.0
            )
            response.raise_for_status()
            return response.json()

# Uso
service = APIService("https://api.example.com", token=os.getenv("API_TOKEN"))
data = await service.get("users/profile")
```

Marp soporta resaltado de sintaxis para **todos los lenguajes** populares.

---

## Citas y alertas

> Las presentaciones son el puente entre una idea técnica compleja y la comprensión del equipo. Marp te permite cruzarlo sin fricción.

<div style="margin-top:1.2rem">

<div class="alerta alerta-info">
<span>ℹ️</span><div><strong>Información:</strong> Las citas usan el bloque <code>&gt;</code> estándar de Markdown con estilos personalizados.</div>
</div>

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div><strong>Correcto:</strong> Los bloques <code>.alerta</code> son componentes HTML reutilizables del tema.</div>
</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div><strong>Atención:</strong> Para usar HTML en Marp debes activar <code>--html</code> en la CLI o en VS Code.</div>
</div>

<div class="alerta alerta-danger" style="margin-top:0.6rem">
<span>🚨</span><div><strong>Error:</strong> No confundas directivas globales con locales — las locales llevan guión bajo: <code>_class</code>.</div>
</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Layouts y Columnas

## Organización visual del contenido

---

## Dos columnas equilibradas

<div class="cols-2">

<div>

### Frontend

<span class="badge badge-blue">React 18</span>
<span class="badge badge-green">TypeScript</span>
<span class="badge badge-purple">Tailwind</span>

- Componentes reutilizables
- Server-Side Rendering con **Next.js**
- Estado global con **Zustand**
- Testing con **Vitest** + Testing Library

</div>

<div>

### Backend

<span class="badge badge-yellow">Python</span>
<span class="badge badge-red">FastAPI</span>
<span class="badge badge-gray">PostgreSQL</span>

- API REST + GraphQL con **Strawberry**
- ORM asíncrono con **SQLAlchemy 2**
- Caché con **Redis**
- Testing con **pytest** + **httpx**

</div>

</div>

---

## Tres columnas — tarjetas

<div class="cols-3" style="margin-top:1.2rem">

<div class="card card-blue">

### Desarrollo

<div class="kpi"><div class="valor">14</div><div class="etiqueta">Sprints</div><div class="sublabel">Mayo — Octubre</div></div>

- Metodología **Scrum**
- Entregas quincenales
- Code review obligatorio

</div>

<div class="card card-green">

### QA

<div class="kpi"><div class="valor">94%</div><div class="etiqueta">Cobertura</div><div class="sublabel">Tests unitarios + E2E</div></div>

- Pipeline **CI automático**
- Tests de regresión
- Staging environment

</div>

<div class="card card-purple">

### Despliegue

<div class="kpi"><div class="valor">0</div><div class="etiqueta">Downtime</div><div class="sublabel">Blue/green deploy</div></div>

- **Kubernetes** en GKE
- Rollback en < 2 min
- Monitorización 24/7

</div>

</div>

---

## Columna ancha + estrecha

<div class="cols-60-40" style="margin-top:0.5rem">

<div>

### Arquitectura del sistema

```
                    ┌──────────────┐
Internet ─────────▶ │ Load Balancer │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  API Gateway  │  Auth · Rate limit
                    └──────┬───────┘
               ┌───────────┼───────────┐
               ▼           ▼           ▼
          ┌────────┐ ┌──────────┐ ┌────────┐
          │ Users  │ │ Products │ │ Orders │
          └────┬───┘ └────┬─────┘ └────┬───┘
               └──────────┼────────────┘
                    ┌──────▼───────┐
                    │  PostgreSQL   │
                    └──────────────┘
```

</div>

<div>

### Características

<div class="alerta alerta-ok">✅ Alta disponibilidad</div>
<div class="alerta alerta-ok" style="margin-top:0.4rem">✅ Escalado horizontal</div>
<div class="alerta alerta-ok" style="margin-top:0.4rem">✅ Tolerancia a fallos</div>
<div class="alerta alerta-warning" style="margin-top:0.4rem">⚠️ Single DB (roadmap)</div>

**Tecnología:**
`GKE` `Istio` `Pub/Sub` `Redis`

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Tablas y Datos

## Comparativas, métricas y KPIs

---

## Tabla comparativa

| Herramienta | Lenguaje | Exportación | Temas | Curva | ¿Recomendado? |
|:------------|:--------:|:-----------:|:-----:|:-----:|:-------------:|
| **Marp** | Markdown | HTML · PDF · PPTX | CSS custom | Baja | ✅ |
| Reveal.js | HTML/MD | HTML | JS/CSS | Media | ✅ |
| Slidev | Vue/MD | HTML · PDF | Vue SFC | Media | ✅ |
| Impress.js | HTML | HTML | CSS | Alta | ⚠️ |
| PowerPoint | GUI | PPTX | Plantillas | Baja | ❌ |
| Google Slides | GUI | Varios | Limitados | Baja | ❌ |

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Para equipos técnicos que usan Git, <strong>Marp</strong> o <strong>Slidev</strong> son las opciones más productivas a largo plazo.</div>
</div>

---

## KPIs del proyecto

<div class="cols-4" style="margin-top:1.2rem">

<div class="card card-blue kpi">
<div class="valor">98.7%</div>
<div class="etiqueta">Uptime</div>
<div class="sublabel">Último trimestre</div>
</div>

<div class="card card-green kpi">
<div class="valor">42ms</div>
<div class="etiqueta">Latencia P95</div>
<div class="sublabel">Tiempo de respuesta</div>
</div>

<div class="card card-yellow kpi">
<div class="valor">12M</div>
<div class="etiqueta">Req / día</div>
<div class="sublabel">Pico: 18M</div>
</div>

<div class="card card-purple kpi">
<div class="valor">4.8★</div>
<div class="etiqueta">Satisfacción</div>
<div class="sublabel">NPS: 72</div>
</div>

</div>

<div style="margin-top:1.4rem">

### Progreso del roadmap

<div class="steps">
<div class="step hecho">Diseño</div>
<div class="step hecho">MVP</div>
<div class="step activo">Beta</div>
<div class="step">GA</div>
<div class="step">v2.0</div>
</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Directivas y Control

## Parámetros globales, locales y clases

---

## Directivas globales

Se definen en el **frontmatter** y aplican a toda la presentación:

```yaml
---
marp: true               # Activa Marp (obligatorio)
theme: profesional       # Nombre del tema CSS
paginate: true           # Muestra número de página
header: 'Mi cabecera'    # Texto del encabezado
footer: 'Mi pie'         # Texto del pie
size: 16:9               # Tamaño: 16:9 | 4:3 | A4 | A0
backgroundColor: #fff    # Color de fondo global
backgroundImage: url()   # Imagen de fondo global
class: lead              # Clase CSS por defecto
---
```

<div class="alerta alerta-info">
<span>ℹ️</span><div>También puedes usar directivas globales desde cualquier diapositiva con <code>&lt;!-- directiva: valor --&gt;</code> (sin guión bajo). Afectan a todas las siguientes.</div>
</div>

---

## Directivas locales

El prefijo `_` limita la directiva **solo a esa diapositiva**:

<div class="cols-2">

<div>

```markdown
<!-- _class: capitulo -->
<!-- _paginate: false -->
<!-- _backgroundColor: #0f172a -->
<!-- _color: white -->
<!-- _backgroundImage: none -->
<!-- _header: '' -->
<!-- _footer: '' -->
```

</div>

<div>

| Directiva | Efecto |
|-----------|--------|
| `_class` | Clase CSS de la sección |
| `_paginate: false` | Oculta el número |
| `_backgroundColor` | Fondo solo aquí |
| `_color` | Color de texto |
| `_backgroundImage` | Imagen local |
| `_header / _footer` | Texto o vacío |

</div>

</div>

---

## Imágenes como fondo

```markdown
<!-- Fondo completo -->
![bg](foto.jpg)

<!-- Con opacidad — útil para texto encima -->
![bg opacity:0.2](foto.jpg)

<!-- Con filtros CSS -->
![bg blur:4px brightness:0.6](foto.jpg)

<!-- Split: imagen a la derecha, contenido a la izquierda -->
![bg right:45%](foto.jpg)

<!-- Split a la izquierda -->
![bg left:35% fit](foto.jpg)

<!-- Imagen inline con dimensiones exactas -->
![w:400px h:250px](diagrama.png)
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Los filtros <code>bg</code> se aplican con directivas de imagen, no con CSS directamente sobre el <code>&lt;img&gt;</code>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">06</p>

# Imágenes

## Fondos, splits, filtros e imágenes inline

---

## Sintaxis de imágenes en Marp

Marp extiende la sintaxis estándar `![alt](src)` con **directivas entre corchetes**:

<div class="cols-2">

<div>

```markdown
<!-- Imagen inline con tamaño fijo -->
![w:300px](foto.jpg)
![h:200px](foto.jpg)
![w:300px h:200px](foto.jpg)

<!-- Imagen de fondo completo -->
![bg](foto.jpg)

<!-- Fondo con tamaño -->
![bg cover](foto.jpg)
![bg contain](foto.jpg)
![bg fit](foto.jpg)
![bg auto](foto.jpg)
![bg 70%](foto.jpg)
```

</div>

<div>

```markdown
<!-- Fondo con filtros -->
![bg opacity:.3](foto.jpg)
![bg blur:5px](foto.jpg)
![bg brightness:.7](foto.jpg)
![bg grayscale:1](foto.jpg)
![bg sepia:1](foto.jpg)

<!-- Fondo split -->
![bg right](foto.jpg)
![bg right:40%](foto.jpg)
![bg left:35%](foto.jpg)

<!-- Múltiples fondos en mosaico -->
![bg](foto1.jpg)
![bg](foto2.jpg)
```

</div>

</div>

---

## Imagen inline — control de tamaño

Coloca imágenes dentro del flujo de texto con dimensiones exactas:

<div class="cols-2" style="margin-top:0.8rem; align-items:center;">

<div>

```markdown
<!-- Solo ancho (alto proporcional) -->
![w:480px](https://picsum.photos/800/500)

<!-- Solo alto -->
![h:200px](https://picsum.photos/800/500)

<!-- Ambas dimensiones -->
![w:300px h:180px](https://picsum.photos/800/500)

<!-- Con texto alternativo descriptivo -->
![Foto de montaña w:400px](foto.jpg)
```

Las imágenes inline respetan el **flujo del documento** y se pueden combinar con texto en columnas.

</div>

<div style="text-align:center">

![w:460px](https://picsum.photos/800/500?random=1)

<p style="font-size:0.7em; color:#94a3b8; margin-top:0.4rem"><code>![w:460px](foto.jpg)</code></p>

</div>

</div>

---

<!-- _backgroundColor: #0f172a -->
<!-- _color: white -->
<!-- _header: '' -->

## Fondo completo — `![bg](foto.jpg)`

![bg](https://picsum.photos/1280/720?random=2)

<div style="background: rgba(15,23,42,0.75); border-radius: 12px; padding: 1.4rem 2rem; max-width: 52%; backdrop-filter: blur(4px); border: 1px solid rgba(255,255,255,0.1);">

### Fondo completo

La imagen ocupa **toda la diapositiva** como fondo. El contenido flota encima.

```markdown
![bg](https://picsum.photos/1280/720)
```

Recomendado usar un **overlay** semitransparente (via `backgroundColor` con alpha o un `<div>` con `rgba`) para que el texto sea legible.

</div>

---

<!-- _backgroundColor: #0f172a -->
<!-- _color: white -->
<!-- _header: '' -->

## Fondo con opacidad — `![bg opacity:.2]`

![bg opacity:.15](https://picsum.photos/1280/720?random=3)

### La imagen queda en segundo plano

Con `opacity` la imagen se mezcla con el `backgroundColor` de la diapositiva, creando un efecto sutil que **no compite con el texto**.

```markdown
<!-- _backgroundColor: #0f172a -->
<!-- _color: white -->

![bg opacity:.15](foto.jpg)
```

- Ideal para diapositivas de capítulo con contexto visual
- Mantiene **legibilidad total** del contenido
- Combinar con `backgroundColor` oscuro o claro según el tono

---

<!-- _color: white -->
<!-- _header: '' -->

## Fondo con filtros — blur + brightness

![bg blur:3px brightness:.45](https://picsum.photos/1280/720?random=4)

<div style="background: rgba(0,0,0,0.45); border-radius:14px; padding:1.6rem 2rem; max-width:55%; border:1px solid rgba(255,255,255,0.15);">

### Filtros disponibles

| Filtro | Ejemplo |
|--------|---------|
| `blur` | `blur:4px` |
| `brightness` | `brightness:.6` |
| `opacity` | `opacity:.3` |
| `grayscale` | `grayscale:1` |
| `sepia` | `sepia:.8` |
| `contrast` | `contrast:1.4` |
| `saturate` | `saturate:1.8` |

Se pueden **combinar**: `![bg blur:3px brightness:.5]`

</div>

---

## Split derecha — `![bg right:40%]`

![bg right:42%](https://picsum.photos/600/800?random=5)

### Layout de imagen + contenido

La directiva `right` parte la diapositiva: el contenido ocupa la izquierda y la imagen la derecha.

```markdown
![bg right:42%](foto.jpg)
```

**Variantes:**

- `![bg right](foto.jpg)` — 50% / 50%
- `![bg right:40%](foto.jpg)` — 60% / 40%
- `![bg left:35%](foto.jpg)` — imagen a la izquierda

Ideal para diapositivas de **producto**, **perfil** o **caso de uso** donde la imagen complementa el texto.

---

## Split izquierda — `![bg left:38%]`

![bg left:38%](https://picsum.photos/600/800?random=6)

<div style="margin-left:1rem">

### Contenido a la derecha

El split izquierdo invierte la composición, útil para crear **variedad visual** a lo largo de la presentación sin repetir el mismo patrón.

```markdown
![bg left:38%](foto.jpg)
```

También puedes combinar `fit` para que la imagen no se recorte:

```markdown
![bg left:38% fit](foto.jpg)
```

Con `fit` la imagen se escala manteniendo proporción y puede dejar márgenes, mientras que sin `fit` (por defecto `cover`) la imagen rellena el área recortándose si es necesario.

</div>

---

## Múltiples fondos — mosaico automático

![bg](https://picsum.photos/640/720?random=10)
![bg](https://picsum.photos/640/720?random=11)
![bg](https://picsum.photos/640/720?random=12)

<!-- _color: white -->
<!-- _header: '' -->

<div style="background:rgba(0,0,0,0.6); border-radius:12px; padding:1.2rem 1.6rem; position:absolute; bottom:60px; left:64px; right:64px;">

### Varios `![bg]` → mosaico horizontal automático

```markdown
![bg](foto1.jpg)
![bg](foto2.jpg)
![bg](foto3.jpg)
```

Marp divide el fondo en columnas iguales, una por imagen. Funciona con cualquier número de imágenes.

</div>

---

## Imagen en columna con texto

<div class="cols-40-60" style="align-items:center; gap:2rem;">

<div style="text-align:center;">

![w:340px](https://picsum.photos/600/400?random=7)

<p style="font-size:0.7em; color:#94a3b8; margin-top:0.5rem">Imagen inline en columna CSS</p>

</div>

<div>

### Imagen + texto con grid

Este layout usa **CSS Grid** (`cols-40-60`) con una imagen inline `![w:340px]` en la columna izquierda.

Es la alternativa al split de fondo cuando necesitas:

- Imagen con **borde redondeado** o sombra
- **Caption** debajo de la imagen
- Imagen que no ocupe todo el alto
- Combinar **varias imágenes** en una columna

```markdown
<div class="cols-40-60">
<div>

![w:340px](foto.jpg)

</div>
<div>

Contenido de texto aquí.

</div>
</div>
```

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">07</p>

# CLI y Exportación

## Comandos, opciones y flujo de trabajo

---

## Marp CLI — Comandos esenciales

```bash
# Instalar
npm install -g @marp-team/marp-cli

# Preview con recarga automática en http://localhost:8080
marp --watch --server presentacion.md

# Exportar a HTML (autónomo, sin dependencias externas)
marp presentacion.md -o salida.html

# Exportar a PDF (requiere Chromium)
marp presentacion.md -o salida.pdf

# Exportar a PowerPoint
marp presentacion.md -o salida.pptx

# Usar tema personalizado
marp presentacion.md --theme ./tema-profesional.css -o salida.html

# Habilitar HTML en el Markdown
marp presentacion.md --html -o salida.html

# Tamaño personalizado
marp presentacion.md --size 1920x1080 -o salida.pdf
```

---

## Configuración por proyecto

Crea `.marprc.yml` en la raíz para no repetir flags:

<div class="cols-2">

<div>

```yaml
# .marprc.yml
html: true
theme: ./tema-profesional.css
size: 16:9
pdf:
  outlines: true
options:
  allowLocalFiles: true
```

</div>

<div>

Ahora basta con:

```bash
# Usa la config automáticamente
marp presentacion.md -o salida.pdf
```

También puedes configurar VS Code:

```json
// .vscode/settings.json
{
  "markdown.marp.themes": [
    "./tema-profesional.css"
  ],
  "markdown.marp.enableHtml": true
}
```

</div>

</div>

---

## Flujo de trabajo recomendado

<div class="cols-3" style="margin-top:1rem">

<div class="card card-blue">

### 1. Editar

<span class="badge badge-blue">VS Code</span>

- Extensión **Marp for VS Code**
- Preview en tiempo real
- `Ctrl+Shift+V` para abrir

</div>

<div class="card card-green">

### 2. Versionar

<span class="badge badge-green">Git</span>

- Solo archivos de texto `.md` y `.css`
- Pull requests con **diff legible**
- Histórico de cambios claro

</div>

<div class="card card-purple">

### 3. Publicar

<span class="badge badge-purple">CI/CD</span>

- GitHub Actions genera PDF/HTML
- Deploy automático a GitHub Pages
- Notificación al equipo

</div>

</div>

<div class="alerta alerta-ok" style="margin-top:1.2rem">
<span>✅</span><div>Con este flujo, la presentación vive en el repositorio del proyecto y se actualiza con cada merge a <code>main</code>.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## ¿Preguntas?

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 josedom24@gmail.com</span>
  <span>🌐 marp.app</span>
  <span>📦 @marp-team/marp-cli</span>
</div>

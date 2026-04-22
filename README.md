# Presentaciones con Marp

Repositorio de presentaciones creadas con [Marp](https://marp.app/), el ecosistema que convierte archivos Markdown en diapositivas HTML, PDF o PowerPoint.

## Contenido

| Archivo | Descripción |
|---|---|
| `presentacion.md` | Presentación de ejemplo sobre Marp: sintaxis, tipografía, layouts y exportación |
| `tema-profesional.css` | Tema CSS personalizado con clases de portada, capítulos y layouts de columnas |
| `.marprc.yml` | Configuración global de Marp (tema, tamaño 16:9, HTML habilitado) |

## Requisitos

- [Marp CLI](https://github.com/marp-team/marp-cli) o la extensión [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode)

```bash
npm install -g @marp-team/marp-cli
```

## Uso

### Previsualizar en VS Code

Instala la extensión **Marp for VS Code** y abre cualquier `.md` con `marp: true` en el frontmatter.

### Exportar desde la terminal

```bash
# HTML
marp presentacion.md -o presentacion.html

# PDF
marp presentacion.md --pdf -o presentacion.pdf

# PowerPoint
marp presentacion.md --pptx -o presentacion.pptx
```

La configuración de `.marprc.yml` se aplica automáticamente al ejecutar `marp` en este directorio.

## Crear una nueva presentación

1. Copia `presentacion.md` como punto de partida.
2. El frontmatter mínimo necesario es:

```markdown
---
marp: true
theme: profesional
---
```

3. Separa las diapositivas con `---`.
4. Usa las clases del tema para layouts especiales: `portada`, `capitulo`, `cols-2`, `cols-60-40`.

## Autor

José Domingo Muñoz · [josedom24@gmail.com](mailto:josedom24@gmail.com)

# Biblioteca — contexto del proyecto

Repositorio personal de Sebastian para registrar palabras nuevas encontradas leyendo libros, más dos páginas complementarias (libros leídos y línea de tiempo de autores). Repo público en GitHub (`stleye/biblioteca`), publicado con GitHub Pages en **https://stleye.github.io/biblioteca/**.

Todo el sitio son 4 páginas HTML autocontenidas (sin build step, sin dependencias externas) que comparten la misma paleta y tipografía:

| Página | Contenido | URL |
|---|---|---|
| `index.html` | Portada del sitio, con tarjetas de navegación a las otras 3 páginas | / |
| `vocabulario.html` | Palabras nuevas, agrupadas por libro, con buscador | /vocabulario.html |
| `libros.html` | Libros leídos, agrupados por autor, con sinopsis, personajes y portada | /libros.html |
| `autores.html` | Línea de tiempo visual de cuándo vivió cada autor | /autores.html |

`index.html` es la página que GitHub Pages sirve en la raíz (tiene prioridad sobre `README.md`). Si se agrega o saca un libro/autor/cantidad de palabras, actualizar a mano los `<span class="room-stat">` de `index.html` (no se calculan dinámicamente, están hardcodeados a propósito para que la portada cargue instantánea sin parsear los otros archivos).

No hay `vocabulario.md` — se eliminó y se migró todo a HTML (agosto 2026).

## Estilo compartido

Las tres páginas usan el mismo bloque de tokens CSS al inicio del `<style>` (paleta "papel/tinta", tema claro y oscuro):

```css
--paper, --paper-raised, --ink, --ink-soft, --rule, --rule-soft,
--accent, --accent-soft, --accent2, --gold, --shadow
```

- Tipografía: Georgia/serif para texto, `ui-monospace` para eyebrows/labels/años.
- Soporta tema claro/oscuro/sistema (`prefers-color-scheme` + `[data-theme]`).
- Si se crea o rediseña una página nueva del sitio, copiar este bloque de tokens tal cual desde `autores.html` para mantener consistencia visual. No inventar una paleta nueva.

## Cómo agregar una palabra nueva

Editar `vocabulario.html`, buscar el arreglo `const libros = [...]` al final del archivo (dentro del `<script>`).

- Si el libro ya existe: agregar un objeto al arreglo `palabras` de ese libro:
  ```js
  { palabra: `nombre`, definicion: `...`, ejemplo: `"..."` }
  ```
- Si el libro no existe todavía: agregar un nuevo objeto al arreglo `libros`:
  ```js
  { titulo: `Título`, autor: `Autor`, palabras: [...] }
  ```
- Usar comillas invertidas (template literals) para los strings, no comillas dobles, porque los ejemplos ya llevan comillas dobles adentro.
- Para dar énfasis dentro de una definición se puede usar `*palabra*` (se convierte a `<em>` automáticamente vía `formatEmphasis`).
- Si el usuario no tiene definición/ejemplo a mano, redactarlos yo mismo (definición tipo diccionario + un ejemplo de uso natural en español).

## Cómo agregar un libro nuevo

Editar `libros.html`, arreglo `const libros = [...]`:

```js
{
  titulo: "Título",
  autor: "Autor",
  sinopsis: "2-4 oraciones, sin spoilers grandes del final si se puede evitar.",
  personajes: ["Personaje 1", "Personaje 2"],
  portada: "URL de la imagen"
}
```

Los libros se agrupan y ordenan alfabéticamente por autor automáticamente en el render (`porAutor` + `sort`) — no hace falta ordenarlos a mano en el arreglo.

**No pedir portada al usuario ni usar placeholders inventados.** Buscar la portada real con la API pública de Open Library:

```bash
rtk proxy curl -s "https://openlibrary.org/search.json?q=$(python3 -c "import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))" "Título Autor")&fields=title,author_name,cover_i&limit=3"
```

Tomar el `cover_i` del resultado que mejor matchea, y construir la URL:
```
https://covers.openlibrary.org/b/id/{cover_i}-L.jpg
```

Verificar que la imagen carga antes de commitear (siguiendo redirects):
```bash
rtk proxy curl -sL -o /dev/null -w "%{http_code}" "https://covers.openlibrary.org/b/id/{cover_i}-L.jpg"
```
Debe devolver `200`.

⚠️ **Importante sobre `curl` en este entorno**: hay un hook/proxy `rtk` que reescribe `curl` normal y devuelve un JSON-schema en vez del body real. Para pegarle a una API externa y obtener la respuesta real, anteponer `rtk proxy`:
```bash
rtk proxy curl -s "https://..."
```

## Cómo agregar un autor a la línea de tiempo

Editar `autores.html`, arreglo `const autores = [...]`:

```js
{
  nombre: "Nombre Apellido",
  nacimiento: 1800,
  muerte: 1870,   // null si sigue vivo
  nacionalidad: "País"
}
```

- **No agregar campo `obra`** — se sacó a pedido explícito del usuario ("no tiene sentido"). Solo nombre, años, nacionalidad.
- La página calcula y muestra sola la edad al morir (`muerte - nacimiento`).
- Si el autor nació antes del año actual de `YEAR_MIN` (actualmente 1630), hay que bajar `YEAR_MIN` para que entre en la escala, y actualizar el texto del `<footer>` que menciona el rango de años.
- Las filas se ordenan solas por año de nacimiento — no hace falta ordenar el arreglo a mano.

## Flujo de trabajo / convenciones

- El repo es **público** y usa **GitHub Pages** (rama `main`, carpeta `/`). Cualquier archivo `.html` en la raíz queda accesible en `https://stleye.github.io/biblioteca/<archivo>.html` unos minutos después del push.
- Actualizar `README.md` cuando se agregan libros nuevos (lista en la sección "## Lecturas") o páginas nuevas (agregar el link).
- Commitear y pushear solo cuando el usuario lo pida explícitamente ("commiteá y pusheá"), salvo que ya haya dado luz verde genérica en la conversación.
- Mensajes de commit en español, describiendo qué se agregó (ej. "Agregar palabras de X al vocabulario", "Agregar autor Y a la línea de tiempo").
- **No** agregar trailer `Co-Authored-By` en los commits (preferencia del usuario, ver memoria `no-coauthor-in-commits`).
- Después de un push, GitHub Pages tarda uno o dos minutos en reflejar el cambio — si hace falta confirmar que ya está online, usar `Monitor` con un loop de `curl` en vez de sondear manualmente.

## Historia relevante (para no repreguntar)

- El vocabulario original vivía en un único `vocabulario.md`, todo de *La caída de la Casa Usher*. Se separó por libro (agosto 2026) y luego se migró completo a `vocabulario.html`.
- `libros.html` empezó como lista plana y se rediseñó para agrupar por autor, con sinopsis y personajes agregados a pedido, y portadas reales buscadas después.
- `autores.html` es la primera página del sitio, sirvió de referencia de estilo para las otras dos.

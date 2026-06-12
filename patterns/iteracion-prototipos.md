# Patrones — Iteración de Prototipos & Datos

Leer este archivo SIEMPRE antes de modificar un prototipo existente o importar datos.

---

## 1. Iteraciones incrementales — cuándo editar y cuándo reescribir

- **< 40% del archivo afectado** → `str_replace` quirúrgico
- **Cambio de layout global** → reescribir el archivo completo
- Siempre `cp` a `/home/claude/` antes de editar (los uploads son read-only)

---

## 2. Marcadores de sección JS — prototipos iterables y seguros

Todo prototipo que contenga bloques de datos grandes (arrays, constantes) **debe incluir
comentarios de sección** como delimitadores. Esto permite reemplazos programáticos seguros
en sesiones futuras sin riesgo de corrupción:

```js
// ── DATA ──
const COLS = [
  { id: 1, title: "Lisboa", ... },
  // ...
];

// ── STATE ──
let activeCol = null;
let activeTag = null;

// ── RENDER ──
function render() { ... }

// ── EVENTS ──
document.addEventListener('DOMContentLoaded', init);
```

Patrón de reemplazo Python usando los marcadores como anclas:

```python
start_marker = "// ── DATA ──\nconst COLS = ["
end_marker   = "];\n\n// ── STATE ──"

start_idx = html.find(start_marker)
end_idx   = html.find(end_marker)

new_html = (
    html[:start_idx]
    + replacement          # nuevo bloque DATA completo
    + "\n\n// ── STATE ──"
    + html[end_idx + len(end_marker):]
)
```

**Regla:** incluir SIEMPRE estos comentarios en prototipos que se vayan a modificar
iterativamente — son la única forma segura de hacer reemplazos programáticos en HTML/JS.

---

## 3. Anti-patrón: parchear template literals JS desde Python

Reemplazar contenido dentro de template literals JS (`` `...${expr}...` ``) desde Python
es extremadamente frágil por los escapes anidados. Produce escapes corruptos silenciosos
que rompen el prototipo sin error evidente.

```python
# ❌ MAL — peligroso, produce escapes corruptos
html = html.replace(
    "${s.img ? `<img src=\"${s.img}\">` : '<div class=\"ph\"></div>'}",
    "'<div class=\"ph\"><svg ...\\\"attr\\\"...></svg></div>'"
)

# ✅ BIEN — localizar la función completa y reescribirla desde cero
new_render = """
function renderSpots(spots) {
  document.getElementById('d-spots').innerHTML = spots.map(s => `
    <div class="sthumb">
      <div class="sthumb-ph"><!-- SVG placeholder --></div>
    </div>
  `).join('');
}
"""
start = html.find('function renderSpots(')
end   = html.find('\n}', start) + 2
html  = html[:start] + new_render + html[end:]
```

**Regla:** ante cualquier modificación de un bloque con template literals JS,
localizar la función completa y reescribirla íntegra — nunca parchear expresiones internas.

---

## 4. Validación de sintaxis JS — usar `node --check`

`new Function(code)` falla con código moderno. Validador correcto:

```bash
node --input-type=module << 'EOF'
import {readFileSync,writeFileSync} from 'fs';
const m = readFileSync('archivo.html','utf8').match(/<script>([\s\S]*?)<\/script>/);
writeFileSync('/tmp/test.js', m[1]);
EOF
node --check /tmp/test.js && echo "SYNTAX OK"
```

Error silencioso típico: un `str_replace` que borra `function foo(){` dejando el cuerpo
flotando. Validar SIEMPRE tras cualquier edición programática.

---

## 5. Flujo CSV → estructura JS agrupada (colecciones de spots)

Patrón para convertir un CSV con filas repetidas (una fila por spot) en una estructura
JS agrupada por colección, lista para incrustar en un prototipo:

```python
from collections import OrderedDict
import csv, json, re

collections = OrderedDict()

for row in rows:
    key = (row['Destino'], row['Tag'])          # clave de agrupación
    if key not in collections:
        collections[key] = {
            'title': row['Título'],
            'img':   row['Cover'],              # ← primera URL = portada canónica (ver abajo)
            'spots': []
        }
    # parsear coordenadas
    m = re.findall(r'-?\d+\.\d+', row['Coordinates'])
    lat = float(m[0]) if len(m) >= 1 else 0.0
    lng = float(m[1]) if len(m) >= 2 else 0.0

    collections[key]['spots'].append({
        'name': row['Spots'].split(',')[0].strip(),  # solo nombre, sin ciudad/país
        'lat':  lat,
        'lng':  lng
    })

# Serializar a JS — SIEMPRE con json.dumps() para tildes, comillas y unicode
for i, (key, col) in enumerate(collections.items()):
    spots_js = json.dumps(col['spots'], ensure_ascii=False)
    print(f"  {{id:{i+1}, title:{json.dumps(col['title'])}, img:{json.dumps(col['img'])}, spots:{spots_js}}},")
```

**Portada canónica:** la imagen de portada de cada colección es la URL de la **primera
fila** del grupo. Las filas siguientes del mismo grupo se ignoran para `img` — incluso si
cambia el parámetro query (`?q=80`, `?q=81`...).

**Regla:** usar siempre `json.dumps()` para strings en el output JS — maneja
automáticamente tildes, comillas y caracteres especiales sin errores de encoding.

# Patrones aprendidos — prototipado Passporter

Patrones extraídos de sesiones reales de prototipado con la skill `ux-prototyper-v2`.
La skill (SKILL.md en Claude) descarga este repo al inicio de cada sesión y lee el
archivo del tema correspondiente **antes** de implementar nada de ese tema.

| Archivo | Tema |
|---|---|
| `drag-drop.md` | Ghost custom, drop indicators, multi-destino, closures stale, threshold click/drag, cross-layer, empty state = drop zone |
| `sidebar.md` | Estructura de navegación, colapsado/expandido, toggle, centrado de iconos, panel central colapsable |
| `layout-paneles.md` | Columnas del itinerario, panel detalle como flex sibling, grids de selector |
| `estado-js.md` | Mapa de estados, estado JS como fuente única, sincronización lista↔ficha, funciones centrales |
| `validacion-errores.md` | Validación en capas, solapamiento de intervalos, infobox de error, mensajes exactos |
| `cards-listas.md` | Click vs checkbox, ficha de spot, bookmark, selector de día, botones persistentes, tag chips, toast con acción, empty state canónico |
| `modales-overlays.md` | Modal-en-modal, first-time flow vs recurrente, z-index |
| `mapas-leaflet.md` | Carga dinámica de Leaflet, guards obligatorios, interacción mapa↔paneles |
| `iteracion-prototipos.md` | str_replace vs reescritura, node --check, marcadores de sección, CSV→JS, anti-patrón template literals |

**Cómo añadir un patrón nuevo:** escribirlo en el archivo del tema que corresponda
(o crear un archivo nuevo y añadirlo a esta tabla y al índice del SKILL.md). Mantener
el formato: título, contexto del bug o decisión, código, y una **Regla** final accionable.

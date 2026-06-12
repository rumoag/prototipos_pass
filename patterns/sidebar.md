# Patrones — Sidebar del viaje

**Fuente de verdad: `preview/sidebar.html` del DS. Leerlo siempre antes de implementar.**
Todos los tokens, medidas exactas, colores y comportamientos están ahí — este archivo
solo recoge la estructura y las trampas conocidas. Si hay duda, el DS gana.

---

## 1. Estructura de navegación — nunca inventarla

Ítems en orden:
```
Logo (header con border-bottom)
Vista general (SquaresFour) · Guardados (Bookmark) · Itinerario (Calendar)
Tareas (ClipboardText) · Tablero (Kanban) · Reservas (File)
── Travel Store ── · Experiencias (Ticket)+New · Ideas (Briefcase) · Imprescindibles (Globe)
Footer: Ayuda · Invitar · Ajustes (border-top)
```

---

## 2. Patrón de implementación correcto

### Estado inicial: colapsado por defecto
```html
<aside class="trip-sidebar col" id="tripSidebar">
```
`.col` = colapsado. Sin clase = expandido.

### Botón toggle — flotante, visible solo en hover
```css
.ts-toggle {
  position: absolute; top: 16px; right: -14px;
  width: 28px; height: 28px; border-radius: var(--radius-full);
  background: var(--bg-default); border: 1px solid var(--border-subtle);
  box-shadow: var(--shadow-sm);
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; color: var(--text-tertiary);
  opacity: 0; pointer-events: none; transition: opacity var(--dur-fast);
}
.trip-sidebar:hover .ts-toggle { opacity: 1; pointer-events: auto; }
```
```html
<button class="ts-toggle" onclick="toggleSidebar()">
  <iconify-icon id="tsToggleIcon" icon="ph:caret-left" style="font-size:14px"></iconify-icon>
</button>
```

### ⚠️ overflow en el sidebar raíz — CRÍTICO
```css
.trip-sidebar { overflow: visible; } /* NUNCA overflow:hidden — recorta el botón toggle */
```
Si se necesita clip en el contenido, aplicarlo a los hijos, no al contenedor raíz.

### Footer — columna en colapsado, fila en expandido
```css
.ts-footer { display: flex; flex-direction: column; gap: 2px; }
.trip-sidebar:not(.col) .ts-footer { flex-direction: row; gap: 4px; }
.trip-sidebar:not(.col) .ts-footer .ts-item-label { display: none; }
```

### Separador Travel Store — visible solo en colapsado
```css
.ts-store-divider { display: none; height: 1px; background: var(--border-subtle); margin: 8px 4px; }
.trip-sidebar.col .ts-store-divider { display: block; }
```

### Nav items — usar `<button>` nativos, nunca `<div>`
```html
<button class="ts-item active" title="Itinerario">
  <div class="ts-item-icon"><iconify-icon icon="ph:map-trifold"></iconify-icon></div>
  <span class="ts-item-label">Itinerario</span>
</button>
```

---

## 3. Sidebar colapsado — centrado de iconos

El padding lateral en el nav rompe el `justify-content: center`. El patrón correcto es
padding cero en el nav y márgenes en el ítem:

```css
/* MAL: padding lateral rompe justify-content:center */
.ts-nav { padding: 14px 14px 0; }

/* BIEN */
.ts-nav { padding-top: 8px; padding-left: 0; padding-right: 0; }
.ts-item { margin: 0 auto; justify-content: center; padding: 0; gap: 0; }
.trip-sidebar:not(.col) .ts-item { justify-content: flex-start; gap: 8px; }

/* Badge fuera del flujo flex en colapsado */
.ts-item .badge-new { position: absolute; right: 6px; opacity: 0; }
.trip-sidebar:not(.col) .ts-item .badge-new { position: static; opacity: 1; margin-left: auto; }
```

Los valores exactos de padding y margin están en `preview/sidebar.html`.

---

## 4. Transición colapsado ↔ expandido

El ancho exacto en cada estado está en `preview/sidebar.html`. El patrón de transición:

```css
.trip-sidebar { transition: width var(--dur-base) var(--ease-standard); overflow: visible; }
.ts-item-label { opacity: 0; max-width: 0; transition: opacity var(--dur-fast); white-space: nowrap; overflow: hidden; }
.trip-sidebar:not(.col) .ts-item-label { opacity: 1; max-width: 160px; }
```

```js
function toggleSidebar() {
  const isCol = document.getElementById('tripSidebar').classList.toggle('col');
  const icon  = document.getElementById('tsToggleIcon');
  if (icon) icon.setAttribute('icon', isCol ? 'ph:caret-right' : 'ph:caret-left');
}
```

Los paneles hermanos que se reposicionan al expandir dependen del layout de cada prototipo
— sus valores de `left` son específicos de cada pantalla, no valores fijos del sidebar.

---

## 5. Panel central colapsable — el mapa crece con CSS puro

```css
.app.panel-closed .panel-add { display: none; }
.app.panel-closed .map-area  { left: var(--sidebar-w); } /* sidebar colapsado */
.app.panel-closed.sidebar-exp .map-area { left: var(--sidebar-expanded); } /* sidebar expandido */
.map-reopen { position: absolute; top: 50%; left: 8px; display: none; }
.app.panel-closed .map-reopen { display: flex; }
```
Nunca usar `hidden` para el colapso del panel — impide que el mapa crezca.
Los valores exactos de `--sidebar-w` y `--sidebar-expanded` vienen de `tokens.css`.

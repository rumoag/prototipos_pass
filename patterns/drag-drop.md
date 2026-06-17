# Patrones — Drag & Drop

Patrones aprendidos en sesiones de prototipado de Passporter. Leer este archivo completo
antes de implementar cualquier drag & drop, kanban o reordenación.

---

## 1. Ghost custom (suprimir el ghost del browser)

```js
const ghost = document.createElement('div');
ghost.className = 'drag-ghost';
document.body.appendChild(ghost);

document.addEventListener('mousemove', e => {
  if (activeDrag) { ghost.style.left=(e.clientX+14)+'px'; ghost.style.top=(e.clientY-18)+'px'; }
});

el.addEventListener('dragstart', e => {
  const c = document.createElement('canvas'); c.width=1; c.height=1;
  e.dataTransfer.setDragImage(c, 0, 0);
  ghost.classList.add('visible');
  requestAnimationFrame(() => el.classList.add('dragging'));
});

el.addEventListener('dragend', () => {
  ghost.classList.remove('visible'); el.classList.remove('dragging');
  document.querySelectorAll('.drop-zone').forEach(z => z.classList.remove('drag-over'));
});

zone.addEventListener('dragleave', e => {
  if (!zone.contains(e.relatedTarget)) zone.classList.remove('drag-over');
});
```

```css
.drag-ghost {
  position:fixed; pointer-events:none; z-index:9999;
  background:var(--bg-default); border:1.5px solid var(--primary-500);
  border-radius:var(--radius-md); padding:8px 12px;
  box-shadow:var(--shadow-lg);
  opacity:0; transition:opacity var(--dur-fast); transform:rotate(1.5deg);
}
.drag-ghost.visible { opacity:1; }
.spot-card.dragging { opacity:0.35; transform:scale(0.97); border-style:dashed; }
.drop-zone.drag-over { border-color:var(--primary-500); background:var(--bg-brand-subtle); transform:scale(1.01); }
```

---

## 2. Drop indicators — medir los items flanqueantes, nunca el indicador

Dos trampas que llevan al mismo error:
- `getBoundingClientRect()` sobre un elemento con `display:none` devuelve `{top:0}`.
- Sobre divs de 3px (los propios indicadores) devuelve valores casi idénticos entre sí.

En ambos casos es imposible distinguir la posición correcta. La solución es medir los
**items reales** que flanquean cada indicator:

```js
function updateDropIndicators(clientY, list) {
  const indicators = [...list.querySelectorAll('.drop-indicator')];
  const items      = [...list.querySelectorAll('.drag-item:not(.dragging)')];

  let bestIdx = 0, bestDist = Infinity;
  indicators.forEach((ind, i) => {
    // Usar el borde del item flanqueante, no el indicador
    const refY = i < items.length
      ? items[i].getBoundingClientRect().top
      : (items.at(-1)?.getBoundingClientRect().bottom ?? 0);
    const dist = Math.abs(clientY - refY);
    if (dist < bestDist) { bestDist = dist; bestIdx = i; }
  });

  indicators.forEach(ind => ind.classList.remove('visible', 'active'));
  indicators[bestIdx].classList.add('visible');          // contexto reorder
  // indicators[bestIdx].classList.add('active');        // contexto drop-en-bloque
  dragState.currentDropIdx = bestIdx;
}
```

**Dos clases CSS para dos contextos:**
```css
.drop-indicator.visible { /* reorder entre spots libres */
  height: 2px; background: var(--primary-500); opacity: 1;
}
.drop-indicator.active { /* spot entrando en bloque horario */
  height: 2px; background: var(--informative-500); opacity: 1;
}
```

Preferir `mousedown/mousemove/mouseup` sobre la API drag nativa.

---

## 3. Drag multi-destino — decidir al soltar, nunca al empezar

El error recurrente es elegir el modo (reordenar vs. mover a bloque) al iniciar el drag.
Siempre que se hace así, uno de los dos comportamientos queda roto.

**Regla:** arrancar el drag igual en todos los casos. Inspeccionar el DOM en `onUp` para
decidir qué acción ejecutar según dónde terminó el cursor.

```js
function onUp(e) {
  if (!dragState.active) return;
  const target = document.elementFromPoint(e.clientX, e.clientY);
  const dropBlock = target?.closest('[data-block-id]');
  const dropDay   = target?.closest('[data-day-id]');

  if (dropBlock) {
    moveSpotToBlock(dragState.spotId, dragState.srcDayId, dragState.srcBlockId, dropBlock.dataset.blockId);
  } else if (dropDay) {
    reorderSpotInDay(dragState.spotId, dragState.srcDayId, dropDay.dataset.dayId, dragState.currentDropIdx);
  }
  // limpiar siempre
  endDrag();
}
```

---

## 4. Closures stale en drag-to-reorder

El bug más persistente: `block` y `list` capturados en el closure se vuelven stale tras
un `renderItinerary()` porque el DOM se recrea y los viejos event listeners apuntan a
nodos huérfanos.

**Regla:** capturar solo IDs primitivos en el closure. Hacer lookup fresco en `onUp`.

```js
// ❌ MAL — captura el nodo vivo, se vuelve stale tras render
el.addEventListener('mousedown', () => {
  dragState = { el, list, block }; // stale tras renderItinerary()
});

// ✅ BIEN — captura solo IDs inmutables
el.addEventListener('mousedown', () => {
  dragState = {
    spotId:   el.dataset.spotId,
    dayId:    el.dataset.dayId,
    blockId:  el.dataset.blockId ?? null
    // lookup fresco de list/block en onUp con querySelector
  };
});

// En onUp:
function onUp(e) {
  const list  = document.querySelector(`[data-day-id="${dragState.dayId}"] .spots-list`);
  const block = dragState.blockId
    ? document.querySelector(`[data-block-id="${dragState.blockId}"]`)
    : null;
  // usar list y block recién obtenidos
}
```

---

## 5. Devolver hijos al padre al borrar un contenedor

Al eliminar un bloque horario, sus spots deben volver a `day.spots` **antes** de borrar
el bloque. Patrón general para cualquier contenedor que agrupe elementos del itinerario.

```js
function deleteBlock(dayId, blockId) {
  const day   = state.days.find(d => d.id === dayId);
  const block = day.blocks.find(b => b.id === blockId);

  // 1. Devolver spots del bloque al array libre del día
  day.spots.push(...block.spots);

  // 2. Ahora sí eliminar el bloque
  day.blocks = day.blocks.filter(b => b.id !== blockId);

  // 3. Re-renderizar
  renderDay(dayId);
}
```

Regla general: **nunca eliminar un contenedor sin primero rescatar sus hijos** al nivel
superior. Aplica a bloques, grupos, secciones o cualquier agrupación de itinerario.

---

## 6. Drag & Drop cross-layer — overlay sobre contenido arrastrable

Cuando hay un drawer o panel con overlay encima de un área con drag & drop, el overlay
bloquea todos los eventos de ratón y el drag queda roto.

**Regla:** el overlay de cualquier drawer debe ser siempre `pointer-events: none` cuando
hay drag & drop activo en el contenido subyacente.

```css
.drawer-overlay {
  pointer-events: none; /* SIEMPRE — nunca bloquear el drag del contenido */
}
/* Si necesitas cerrar el drawer al hacer click fuera, usar el propio drawer */
.drawer { pointer-events: auto; }
```

Para detectar la columna target con `elementFromPoint` cuando hay un ghost flotante:

```js
function getDropTarget(e) {
  // 1. Ocultar el ghost temporalmente para que no interfiera
  ghost.style.display = 'none';

  // 2. elementsFromPoint (plural) devuelve todos los elementos apilados
  const els = document.elementsFromPoint(e.clientX, e.clientY);

  // 3. Restaurar el ghost
  ghost.style.display = '';

  // 4. Filtrar — coger el primer elemento que sea una columna válida
  //    y que NO esté dentro del drawer (para cross-layer)
  return els.find(el =>
    el.matches('.kanban-col') && !el.closest('.drawer')
  ) ?? null;
}
```

---

## 7. Click vs drag en cards — threshold de 5px

Toda card interactiva en Passporter puede tener ambos comportamientos: abrir detalle (click)
y reordenar (drag). Sin threshold, cualquier click mínimamente impreciso activa el drag.

**Implementación con threshold y flag `wasDragged`:**

```js
card.addEventListener('mousedown', e => {
  const startX = e.clientX, startY = e.clientY;
  let dragging = false;

  function onMove(e) {
    const dx = e.clientX - startX, dy = e.clientY - startY;
    if (!dragging && Math.hypot(dx, dy) > 5) {
      dragging = true;
      card.dataset.wasDragged = 'true';
      startDrag(card, e);
    }
  }
  function onUp() {
    document.removeEventListener('mousemove', onMove);
    document.removeEventListener('mouseup', onUp);
    // Limpiar el flag tras un pequeño delay para que el click handler lo vea
    setTimeout(() => delete card.dataset.wasDragged, 50);
  }
  document.addEventListener('mousemove', onMove);
  document.addEventListener('mouseup', onUp);
});

card.addEventListener('click', e => {
  if (card.dataset.wasDragged) return; // ignorar si hubo drag real
  openCardDetail(card.dataset.id);
});
```

---

## 8. Empty state = drop zone — mismo elemento, no duplicar

El empty state de una columna vacía es la drop zone. No crear dos elementos separados.
Un único elemento con clases de estado que se gestiona con CSS:

```html
<div class="col-empty-zone drop-zone" data-col-id="col-1">
  <iconify-icon icon="ph:plus-circle-light"></iconify-icon>
  <span>Arrastra aquí</span>
</div>
```

```css
/* Visible cuando la columna está vacía */
.col-empty-zone { display: flex; /* ... */ }

/* Oculto en cuanto hay cards */
.kanban-col.has-cards .col-empty-zone { display: none; }

/* Estado drag-over */
.col-empty-zone.drag-over {
  border-color: var(--primary-500);
  background: var(--bg-brand-subtle);
}
```

```js
function renderCol(colId) {
  const col   = state.cols.find(c => c.id === colId);
  const colEl = document.querySelector(`[data-col-id="${colId}"]`);
  colEl.classList.toggle('has-cards', col.cards.length > 0);
  // re-renderizar cards...
}
```

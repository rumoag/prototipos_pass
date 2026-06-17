# Patrones — Estado JS

---

## 1. Convertir criterios de aceptación en mapa de estados antes de prototipar

```
Estado A: Sin datos → empty state
Estado B: Con datos → vista principal
Estado C: Modal → overlay encima de B
Estado D: Sub-modal → overlay encima de C
Estado E: Confirmación → toast + actualización de B
```

Cuando un elemento puede vivir en múltiples contenedores, añadir estados de transición.
Este mapa es el Paso 2 obligatorio de la Fase 2 — hacerlo SIEMPRE antes de escribir código.

---

## 2. Estado JS como fuente de verdad única para listas mixtas

```js
const state = {
  freeSpots: { day1:[], day2:[{id,name,emoji}], day3:[] },
  blocks:    { day1:[], day2:[{id,start,end,spots:[]}], day3:[] }
};
```

- **Nunca leer del DOM** para saber dónde está un elemento
- `renderDay()` siempre rebindea event listeners tras re-renderizar
- Al eliminar un contenedor, sus elementos vuelven a `freeSpots`
  (ver `patterns/drag-drop.md` → "Devolver hijos al padre")

---

## 3. Sincronización de estado entre lista y ficha

```js
function saveDtTag() {
  spot.tagIds = pendingTagId === -1 ? [] : [pendingTagId];
  renderDtPill(spot); renderG();
}
function saveAsgn() {
  spot.tagIds = [...pendingAssign];
  renderG();
  if (openDetailId === spot.id) renderDtPill(spot);
}
```

Toda mutación de un dato compartido debe re-renderizar **todas** las superficies donde
aparece (lista, ficha abierta, badges).

---

## 4. Estado "en itinerario" como función central

`spotInAnyDay` debe conocer **todos los contenedores posibles** — spots libres Y spots
dentro de bloques. Si no, el botón `+` de Explorar no se actualiza correctamente al
asignar un spot a un bloque.

```js
function spotInAnyDay(spotId) {
  return state.days.some(day => {
    // Buscar en spots libres del día
    if (day.spots.some(s => s.id === spotId)) return true;
    // Buscar dentro de cada bloque del día
    return day.blocks.some(block =>
      block.spots.some(s => s.id === spotId)
    );
  });
}

// Uso — actualizar el botón + de cada card en Explorar
function refreshExploreBtns() {
  document.querySelectorAll('[data-spot-id]').forEach(card => {
    const inDay = spotInAnyDay(card.dataset.spotId);
    card.querySelector('.add-btn').classList.toggle('in-itinerary', inDay);
  });
}
// Llamar tras cualquier mutación del estado del itinerario
```

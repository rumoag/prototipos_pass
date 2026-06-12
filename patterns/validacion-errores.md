# Patrones — Validación & Errores

---

## 1. Validación en capas: inline vs. al guardar vs. toast

| Nivel | Cuándo | Dónde |
|---|---|---|
| **Inline on-change** | Error lógico simple (fin ≤ inicio, vacío) | Debajo del input, bloquea CTA |
| **Inline al guardar** | Requiere contexto externo (solapamiento) | Debajo del input, bloquea CTA |
| **Toast de error** | Fallo API / operación irreversible | Toast rojo, esquina inferior, 4.5s |

CTA permanece deshabilitado mientras haya error inline activo.

---

## 2. Validación de solapamiento de intervalos

Algoritmo: dos intervalos [A,B] y [C,D] se solapan si `A < D && B > C`.

```js
function hasOverlap(startMin, endMin, dayId, excludeBlockId = null) {
  const day = state.days.find(d => d.id === dayId);
  return day.blocks.some(block => {
    if (block.id === excludeBlockId) return false; // ignorar el propio bloque en modo edición
    return startMin < block.endMin && endMin > block.startMin;
  });
}

// Uso al guardar
function saveBlock(dayId, blockId) {
  const isEdit = !!blockId;
  if (hasOverlap(startMin, endMin, dayId, isEdit ? blockId : null)) {
    showInlineError('errorTime', `Se solapa con otro bloque existente`);
    return;
  }
  // ... guardar
}
```

Casos de error a validar **en orden** (el primero que falla corta la cadena):
1. Hora inicio vacía
2. Hora fin vacía
3. `endMin <= startMin` (fin no posterior a inicio)
4. Solapamiento con otro bloque del mismo día

---

## 3. Patrón visual de errores en modales de tiempo

Los 5 casos de error con sus mensajes exactos y el componente infobox de Atlas:

| Caso | Mensaje |
|---|---|
| Inicio vacío | "Selecciona una hora de inicio" |
| Fin vacío | "Selecciona una hora de fin" |
| Fin ≤ Inicio | "La hora de fin debe ser posterior a la de inicio" |
| Solapamiento | "Este horario se solapa con «[nombre bloque]» ([HH:MM]–[HH:MM])" |
| Sin nombre | "Añade un nombre al bloque" |

**Componente infobox Atlas** (error inline en modales):
```css
.infobox-error {
  background: var(--negative-25); /* #fdf3f1 */
  border: 1px solid var(--negative-500);
  border-radius: var(--radius-md);
  padding: 8px 12px;
  font-size: 13px;
  color: var(--negative-500);
  display: flex;
  align-items: flex-start;
  gap: 6px;
  margin-top: 6px;
}
```
```html
<div class="infobox-error" id="errorTime" style="display:none">
  <iconify-icon icon="ph:warning-circle" style="font-size:16px;flex-shrink:0;margin-top:1px"></iconify-icon>
  <span id="errorTimeMsg"></span>
</div>
```
```js
function showTimeError(msg) {
  const box = document.getElementById('errorTime');
  document.getElementById('errorTimeMsg').textContent = msg;
  box.style.display = 'flex';
  document.getElementById('btnSaveBlock').disabled = true;
}
function clearTimeError() {
  document.getElementById('errorTime').style.display = 'none';
  document.getElementById('btnSaveBlock').disabled = false;
}
// Llamar clearTimeError() siempre al abrir el modal (reset de estado)
```

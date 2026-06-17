# Patrones — Cards, Listas, Tags & Toasts

---

## 1. Interacciones compuestas en card (click vs checkbox)

- **Click en el cuerpo** → navega al detalle / abre modal
- **Click en el checkbox** → selecciona para acción múltiple (sin navegar)

```html
<div class="card" onclick="openDetalle('id')">
  <div class="checkbox" onclick="event.stopPropagation(); toggleSelect('id')">...</div>
</div>
```
Documentar en Fase 1 siempre que haya cards seleccionables.

---

## 2. Ficha de detalle de spot (Figma `2018-17220`)

```
Cover 210px (bookmark top-left + X top-right + dots)
Nombre (22px Heavy, ls -0.55px)
Rating · Categoría
[+ Añadir (bg-inverse)] [↗ Como llegar (outlined)]
Descripción + "Más información"
Etiqueta — pill single-select (vacío→"Nuevo" / con tag→color+nombre)
Info rows: Dirección · Contacto · Web · Horario
```
**Pill ficha = single-select. Chip lista = multi-select.**
Referencia visual completa: `preview/spot-detail.html` del DS.

---

## 3. Bookmark toggle — dos SVG paths distintos

```js
const SVG_OUTLINE = '<path d="M184,32H72..."/>';
const SVG_FILL    = '<path d="M184,32H72...Z"/>';
function updateBmIcon(el, saved) {
  el.style.background = saved ? 'var(--bg-inverse)' : 'rgba(255,255,255,.92)';
  el.querySelector('svg').innerHTML = saved ? SVG_FILL : SVG_OUTLINE;
  el.querySelector('svg').style.color = saved ? '#fff' : 'var(--primary-500)';
}
```
Sincronizar en tres superficies: card Explorar · card Guardados · ficha detalle.
(Los blancos sobre cover de imagen son la excepción aceptada a la regla de tokens.)

---

## 4. Selector de día — dropdown inline relativo al botón

```js
function openCardDaySel(spotId, btnEl, e) {
  e.stopPropagation();
  const dd = document.createElement('div');
  dd.className = 'card-day-dd';
  btnEl.style.position = 'relative';
  btnEl.appendChild(dd);
}
```
Botón alterna entre `+` (sin itinerario) e icono editar (ya en itinerario).

---

## 5. Día activo — sincronizar con acordeón

```js
function toggleDay(i) {
  dayOpen[i] = !dayOpen[i];
  if (dayOpen[i]) { activeDay=i; updatePaDate(DAYS[i]); }
  else {
    const other = dayOpen.findIndex((o,idx) => o && idx!==i);
    if (other !== -1) { activeDay=other; updatePaDate(DAYS[other]); }
    else { activeDay=null; showNoDaySelected(); }
  }
}
```
Cuando `activeDay === null`, el botón `+` abre el selector de día.

---

## 6. Botones de acceso a features — siempre persistentes

En Passporter, los botones de acceso a features secundarias (ej. "Usar plantilla",
"Importar", "Ver sugerencias") **nunca se ocultan** en función del estado del contenido.

**Regla:** antes de implementar cualquier lógica de hide/show en un botón, preguntar
explícitamente al product trio:
> *"¿Este botón es persistente (siempre visible) o condicional (depende del estado)?"*

Por defecto asumir **persistente**. Un botón que desaparece cuando hay contenido
elimina el acceso a la feature para usuarios con datos.

```js
// ❌ MAL — el botón desaparece cuando hay tareas
function renderTasks() {
  btnTemplate.style.display = tasks.length > 0 ? 'none' : 'flex';
}

// ✅ BIEN — siempre visible
function renderTasks() {
  // btnTemplate no se toca
}
```

---

## 7. Tag chips con filtro — estado seleccionado con stroke + ×

Los chips de tag aparecen **solo** en la fila de filtros encima de la lista — nunca como
título de grupo repetido dentro de la lista ni dentro de cada card.

- Estado seleccionado: borde del color propio del chip + `×` al final del texto
- Chips no seleccionados se atenúan con `opacity: 0.38`
- El botón `···` aparece solo cuando hay un filtro activo, a la derecha de la fila

```css
.tag-chip {
  display: inline-flex; align-items: center; gap: 5px;
  height: 32px; padding: 0 12px; border-radius: var(--radius-full);
  font-size: 12px; font-weight: 700; cursor: pointer;
  border: 1.33px solid transparent;
  transition: opacity var(--dur-fast), border var(--dur-fast);
}
/* Colores de tag = tokens de la paleta semántica Atlas */
.tag-chip.green          { background: var(--positive-50); color: var(--positive-700); }
.tag-chip.green.selected { border-color: var(--positive-700); }
.tag-chip.amber          { background: var(--warning-50); color: var(--warning-700); border-color: var(--warning-700); }
.tag-chip.inactive       { opacity: 0.38; }
.tag-chip-x              { font-size: 13px; font-weight: 400; opacity: .7; }
```

```js
// Render chip con estado activo
`<div class="tag-chip ${color}${isActive ? ' selected' : ''}${isInactive ? ' inactive' : ''}"
     onclick="filterByTag('${t.short}')">
  ${t.short}${isActive ? '<span class="tag-chip-x">×</span>' : ''}
</div>`

// ··· solo cuando hay filtro activo
${activeTagFilter ? `
  <div class="saved-dots" onclick="toggleTagMenu(this)" style="margin-left:auto;position:relative">
    ···
    <div class="tag-menu" id="tagMenu" style="display:none">
      <div class="tag-menu-item">Crear nuevo tag</div>
      <div class="tag-menu-item">Editar tag</div>
      <div class="tag-menu-item danger" onclick="deleteActiveTag()">Eliminar tag</div>
    </div>
  </div>` : ''}
```

**Variante sin flags:** cuando el usuario pide solo texto en los chips de destino,
eliminar la referencia al emoji/flag del render. Mantener el objeto `DEST_FLAGS` si existe
para no romper referencias futuras, simplemente no renderizarlo:

```js
// Con flag (por defecto)
`<span class="dest-flag">${DEST_FLAGS[d]||'🌍'}</span>${d}`

// Sin flag (cuando el usuario lo pide explícitamente)
`${d}`
```

---

## 8. Toast con acción directa + badge numérico en tab de destino

El toast de confirmación incluye: ✓ icono + mensaje + separador vertical + acción clickable
que navega directamente al destino sin cerrar nada manualmente.

```html
<div class="toast" id="toast">
  <iconify-icon icon="ph:check-circle" style="font-size:18px;flex-shrink:0"></iconify-icon>
  <span id="toast-msg"></span>
  <div class="toast-sep"></div>
  <div class="toast-action" onclick="goToSaved()">
    Ver guardados
    <iconify-icon icon="ph:arrow-right" style="font-size:14px"></iconify-icon>
  </div>
</div>
```

```css
/* Blancos/transparencias sobre el fondo oscuro del toast: excepción aceptada a tokens */
.toast-sep { width: 1px; height: 16px; background: rgba(255,255,255,.25); flex-shrink: 0; }
.toast-action {
  font-size: 13px; font-weight: 700;
  color: rgba(255,255,255,.85); cursor: pointer;
  display: flex; align-items: center; gap: 4px;
  transition: color var(--dur-fast);
}
.toast-action:hover { color: #fff; }
```

Badge numérico en la tab de destino — actualizar en cada `addToSaved`/`removeFromSaved`:
```js
function updateSavedBadge() {
  const badge = document.getElementById('saved-badge');
  if (!badge) return;
  badge.style.display = savedItems.length === 0 ? 'none' : 'inline-flex';
  badge.textContent = savedItems.length;
}
```

Navegación desde el toast — cierra lo que esté abierto, cambia de tab y oculta el toast:
```js
function goToSaved() {
  if (cur) closeDetail();
  closeSpotDetail();
  switchTab('sav', tabSavedBtn);
  document.getElementById('toast').classList.remove('show');
}
```

---

## 9. Empty state canónico para spots sin imagen (Atlas DS)

Cuando los spots no tienen imagen, el placeholder correcto usa tokens Atlas oficiales:
`--neutral-500` (`#abb2ba`) para la silueta del paisaje y `--neutral-300` (`#cdd1d6`) para el sol.

```html
<div class="sthumb-ph">
  <svg viewBox="0 0 56 56" xmlns="http://www.w3.org/2000/svg"
       style="width:28px;height:28px;opacity:.35">
    <path d="M8 42 L22 22 L32 34 L39 26 L48 42Z" fill="var(--neutral-500)"/>
    <circle cx="41" cy="16" r="5" fill="var(--neutral-300)"/>
  </svg>
</div>
```

```css
.sthumb-ph {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, var(--neutral-100), var(--neutral-200));
  display: flex;
  align-items: center;
  justify-content: center;
}
```

**Regla:** este es el empty state canónico de spots en Passporter. No inventar
placeholders alternativos ni usar colores fuera de la escala Neutral de Atlas.

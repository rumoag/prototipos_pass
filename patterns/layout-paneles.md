# Patrones — Layout & Paneles

---

## 1. Layout de paneles apilados — itinerario (columnas absolutas, 1440px)

| Columna | left | ancho |
|---|---|---|
| Sidebar | 0 | 64px |
| Panel itinerario | 64px | 352px |
| Panel add/detail | 416px | 408px |
| Mapa | 824px | resto |

```css
.panel-itinerary { position:absolute; top:0; left:64px; width:352px; bottom:0; }
.panel-add       { position:absolute; top:0; left:416px; width:408px; bottom:0; z-index:8; }
.panel-detail    { position:absolute; top:0; left:416px; width:408px; bottom:0; z-index:9; }
.map-area        { position:absolute; top:0; left:824px; right:0; bottom:0; }
```

---

## 2. Panel de detalle como tercer panel flex — nunca overlay absoluto sobre el mapa

En layouts sidebar + panel + mapa, el detalle de un ítem debe abrirse como **tercer panel
flex sibling** que empuja el mapa — NO como `position:absolute` encima de él.

**DOM order obligatorio:**
```html
<body> <!-- display:flex; height:100vh; overflow:hidden -->
  <aside class="sb"></aside>          <!-- 64px, flex-shrink:0 -->
  <div class="panel"></div>           <!-- 419px, flex-shrink:0 -->
  <div class="spot-detail" id="spotDetail"></div>  <!-- 0→419px, flex-shrink:0 -->
  <div class="mwrap"></div>           <!-- flex:1, position:relative -->
</body>
```

```css
.spot-detail {
  width: 0; min-width: 0;
  height: 100vh;
  background: var(--bg-default);
  border-left: 1px solid var(--border-default);
  display: flex; flex-direction: column;
  overflow: hidden; flex-shrink: 0;
  transition: width var(--dur-slow) var(--ease-standard),
              min-width var(--dur-slow) var(--ease-standard);
}
.spot-detail.on { width: 419px; min-width: 419px; }
```

```js
function openSpotDetail(spot) {
  document.getElementById('spotDetail').classList.add('on');
  if (map && spot.lat && spot.lng) map.flyTo([spot.lat, spot.lng], 16, { duration: 0.8 });
  hiPin(spot.id);
}
function closeSpotDetail() {
  document.getElementById('spotDetail').classList.remove('on');
  resetPins();
}
```

**Trampas frecuentes:**
- ❌ `position:absolute; transform:translateX(100%)` — queda encima del mapa en lugar de empujarlo
- Leaflet se redimensiona solo cuando `.mwrap` cambia de ancho (`flex:1`). No hace falta `map.invalidateSize()`
- Verificar que `spot-detail` sea **hermano directo** de `.panel` y `.mwrap` — nunca anidado dentro de `.mwrap`
- En prototipos con muchas ediciones iterativas el bloque puede acabar después de `</body>` — extraerlo y reinsertarlo manualmente

---

## 3. Grid de selector — `repeat(3, 1fr)`, nunca flex-wrap con ancho fijo

El selector de plantillas de Passporter usa `grid-template-columns: repeat(3, 1fr)`.
Las cards no tienen ancho fijo — se adaptan al contenedor.

```css
/* ✅ CORRECTO */
.template-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

/* ❌ MAL — ancho fijo rompe el layout en contenedores estrechos */
.template-grid {
  display: flex;
  flex-wrap: wrap;
}
.template-card { width: 160px; }
```

**Regla general:** verificar siempre en el Figma si el layout de una grid es
`repeat(N, 1fr)` o tiene columnas de ancho fijo antes de implementar. Inspeccionar
el frame del componente, no solo la card individual.

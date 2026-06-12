# Patrones — Mapas (Leaflet)

Referencia visual de pins: `preview/map-markers.html` del DS — leerlo siempre antes de
implementar marcadores.

---

## 1. Carga dinámica y guards obligatorios

Leaflet se inyecta dinámicamente para no bloquear el render inicial. **Nunca asumir que
`L` está disponible** antes del callback `onload`. Todas las funciones que accedan al mapa
o a los layer groups deben empezar con un guard.

```js
// Carga dinámica
(function loadLeaflet() {
  const css = document.createElement('link');
  css.rel = 'stylesheet';
  css.href = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css';
  document.head.appendChild(css);

  const js = document.createElement('script');
  js.src = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js';
  js.onload = initMap; // callback — nunca usar L antes de este punto
  js.onerror = () => { /* fallback visual */ };
  document.body.appendChild(js);
})();

// Guard obligatorio en TODA función que use mapa o layer groups
function renderPins() {
  if (!map || !allLG) return; // ← sin esto → "L is not defined" / "clearLayers is not a function"
  allLG.clearLayers();
  // ...
}
```

Errores típicos sin guard: `L is not defined`, `clearLayers is not a function`,
`Cannot read properties of null` — todos causados por llamar funciones de mapa
antes de que Leaflet haya terminado de cargar.

---

## 2. Interacción mapa ↔ paneles

- El detalle de un spot abre como panel flex sibling que empuja el mapa
  (ver `patterns/layout-paneles.md`). Leaflet se redimensiona solo cuando su wrapper
  `flex:1` cambia de ancho — no hace falta `map.invalidateSize()`.
- Al abrir un detalle con coordenadas: `map.flyTo([lat, lng], 16, { duration: 0.8 })`
  y resaltar el pin correspondiente. Al cerrar, resetear pins.

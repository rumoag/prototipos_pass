# Patrones — Modales & Overlays

Referencia visual: `preview/modal.html` del DS — modal app (430px) y web (534px).

---

## 1. Patrón modal-en-modal (navegación interna entre overlays)

```js
function openDetalle(id) {
  document.getElementById('overlay-lista').classList.add('hidden');
  document.getElementById('overlay-detalle').classList.remove('hidden');
}
function volverALista() {
  document.getElementById('overlay-detalle').classList.add('hidden');
  document.getElementById('overlay-lista').classList.remove('hidden');
}
```

Botón "Volver" intercambia visibilidad. Botón X cierra todo.

---

## 2. First-time flow vs flujo recurrente — capas separadas

El onboarding y el flujo recurrente son implementaciones distintas aunque muestren
el mismo contenido del Figma.

| | First-time flow | Flujo recurrente |
|---|---|---|
| **z-index** | `2000+` | `200` |
| **Contenedor** | Modal full-screen | Drawer lateral |
| **Navegación interna** | Swap de vistas con `.hidden`/`.active` | No aplica |
| **Disparador** | Primera vez que entra a la sección | Click en botón persistente |

```css
/* First-time — por encima de todo */
.onboarding-modal { z-index: 2000; }
.onboarding-overlay { z-index: 1999; }

/* Drawer recurrente — por encima del contenido, por debajo del onboarding */
.drawer { z-index: 200; }
.drawer-overlay { z-index: 199; pointer-events: none; }
```

Navegación interna en el first-time (swap de vistas, no drawer):
```js
function showOnboardingStep(step) {
  document.querySelectorAll('.ob-view').forEach(v => v.classList.remove('active'));
  document.getElementById(`ob-${step}`).classList.add('active');
}
// Flujo: showOnboardingStep('whats-new') → showOnboardingStep('selector') → showOnboardingStep('detail')
```

> ⚠️ Si bajo el drawer hay contenido con drag & drop, el overlay debe ser
> `pointer-events: none` — ver `patterns/drag-drop.md` → "Cross-layer".

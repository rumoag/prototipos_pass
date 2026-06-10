# Atlas Design System — Passporter

The internal design system for **Passporter**, a travel-planning app for organizing trips, ideas, activities, and travel inspiration. The project is bilingual / Spanish-first (the source Figma is in Spanish: *Botones*, *Iconografía*, *Modales*, *Tipografía*) and the brand identity ("Atlas") frames the product as a calm, modern travel companion.

This is a working WIP. Source pages: Cover, Tokens-Foundations, Colores, Tipografía, Iconos, Botones, Modales, Toggle, Sidebar, Navegación, Tooltip, Toast, More, Brand, Logo, Playground.

## Sources

- **Figma** — *Atlas - WIP.fig* (mounted as VFS at `/`). Pages of interest: `/Logo`, `/Colores`, `/Tipografia`, `/Iconos`, `/Botones`, `/Modales`, `/Sidebar`, `/Navegaci-n`, `/Toggle`, `/Tooltip`, `/Toast`.
- **Figma — Phosphor Icons** (the icon system referenced by Atlas): `https://www.figma.com/design/XdqMEZypMrscXeVh2fxsIz/Phosphor-Icons` — Atlas uses Phosphor `Light` & `Thin` weights as the canonical product icon set.
- **Tokens** — `tokens.json` (W3C-style design-tokens export from Figma Tokens / Tokens Studio).
- **Fonts** — Cerebri Sans family (Light / Book / Regular / Medium / SemiBold / Bold / Heavy) bundled in `fonts/`.

## Index — what's in this folder

| Path | Purpose |
| --- | --- |
| `README.md` | This file. Brand context, foundations, copywriting, visual rules. |
| `SKILL.md` | Cross-compatible Agent Skill manifest. |
| `tokens.json` | Raw design tokens exported from Figma. |
| `tokens.css` | All color + type CSS variables and semantic classes. **Import this in any Atlas artifact.** |
| `fonts/` | CerebriSans .ttf files. |
| `assets/` | Logos (Eucalipto / Terracota / White, logotype + symbol). |
| `preview/` | One small HTML card per design-system concept (renders into the Design System tab). |
| `ui_kits/passporter-app/` | UI kit: Passporter mobile/web product surfaces (sidebar, modal, buttons, navigation). |
| `ui_kits/passporter-app/index.html` | Demo screen — what a typical Passporter view looks like. |

---

## Brand at a glance

- **One type family.** Atlas runs entirely on **Cerebri Sans**. The display/marketing role uses the heaviest cut (**Heavy / 800**) for hero titles like *Logo*, *Tipografia*, *Botones*; the product role uses Regular→Bold for UI, body, labels and links. There is no separate serif display face.
- **Color personalities.** *Eucalipto* (deep green `#007a51` → `#1b352c`) is the primary brand color — calm, forest-y, the trip-planner color. *Terracota* (warm orange `#e66000`) is the secondary accent — calls-to-action, active states. Plus full neutral, positive, negative, warning and informative scales (each 25→950).
- **Iconography is Phosphor.** Atlas standardizes on the Phosphor Icons library — usually `Light` weight (1.5 stroke), occasionally `Thin`. SVG only.
- **Quiet, photo-led.** Modals embed a single rounded-12 hero image. Cards keep large rounded corners (16px) and minimal chrome. Travel imagery is warm, natural, real.

---

## CONTENT FUNDAMENTALS

### Language
Atlas is **Spanish-first**. Source labels include *Ideas de viaje*, *Actividades*, *Imprescindibles*, *Paquete de viaje*, *Inicio*, *Mis Viajes*, *Explorar*. Translations are provided where needed but the system is designed in Spanish.

### Tone
- **Inviting, not corporate.** Helpful and human — like a well-traveled friend.
- **Concrete and concise.** Sentences in product UI are short. Onboarding and modal copy from the Figma is descriptive but not flowery.
- **Plural collaborative.** Marketing copy uses "nosotros" ("…nos enfocamos…", "…trabajamos con SVG…"). Product copy uses **tú** (informal "you" — never "usted").
- **Functional brand voice.** From the Figma's own Iconografía description: *"Los iconos cumplen una función principalmente comunicativa y funcional: refuerzan acciones, facilitan la comprensión y acompañan el contenido sin generar ruido visual."* The principles named: **claridad, simplicidad, accesibilidad** (clarity, simplicity, accessibility).

### Casing
- **Sentence case for everything in product** — buttons, titles, menu items. ("Ideas de viaje", not "IDEAS DE VIAJE" or "Ideas De Viaje".)
- **Title-case acceptable for marketing display headlines** if needed for visual rhythm, but not the default.
- **No ALL CAPS** — the Label-XSmall style (10px, semibold) keeps casing untouched. Atlas avoids the SaaS overline-uppercase trope.

### Examples (from Figma)
- Section title: **"Tipografia"** (not "Sistema de tipografía")
- Modal stub: **"Title"** + **"Description"** + **"Button"** — utilitarian placeholders.
- Sidebar items: **"Ideas de viaje"**, **"Actividades"**, **"Imprescindibles"** — plural nouns, lowercase after first word.
- Footer: just a year — **"2025"**. No marketing tagline.

### Emoji
**Not used** in product UI. Phosphor icons cover the same role (status, category, action). Travel content may include emoji in user-generated text but the system itself never uses them.

### Punctuation / numbers
- Use Spanish quote marks « » when quoting in marketing copy; straight quotes in product strings are fine.
- Numbers without thousands separators in short product strings (counters, badges).

---

## VISUAL FOUNDATIONS

### Color motifs
- **Primary action surface:** Eucalipto (`primary-500`) for filled CTAs in marketing; **Neutral-950 / `#1b352c`** (a deep eucalipto night) for primary product CTAs — most filled buttons in the kit are dark green-black.
- **Accent / energy:** Terracota for promotional moments, badges, "new" indicators.
- **Surfaces:** Mostly white (`#ffffff`) with `neutral-25` (`#f8f9fa`) for headers/nav strips and subtle inset cards. Inverse surfaces use `#1b352c`.
- **Text default** is `neutral-950` (`#2a2d31`) — never pure black for body. Secondary text is `neutral-800`.

### Type
- **Marketing/display:** Cerebri Heavy (800), 48 / 32 / 24 px, letter-spacing −2%.
- **Product titles:** Cerebri Bold, 20 / 18 / 16, ls −2% (titles), 0% (T3).
- **Body:** Cerebri Regular, 18 / 16 / 14 / 12, ls 0%.
- **Label/Link:** Cerebri SemiBold, 16 / 14 / 12 / 10, ls +1%.
- **Line-heights:** 19, 22, 26, 29, 32, 38, 53 (a discrete scale).

### Spacing & layout
- 4-px base. Common gaps: 4, 8, 12, 16, 20, 24, 32, 48, 64, 80, 120.
- **Page header** (marketing/spec frames): 1888-wide white card, padded 16px from viewport edge, with a 80-px tall `neutral-25` strip carrying the logo on the left. Body content padded 80px from sides.
- **Product canvases** sit on white, content typically left-aligned with max-width ~720–960.

### Backgrounds
- **No gradients in product.** No noise textures. Solid colors.
- **Photography is warm, natural, daylight** — travel photos: landscapes, plates of food, neighborhoods. Imagery is full-bleed inside `radius-lg` (12px) containers within modals. No black-and-white treatments.
- **Hero/marketing surfaces** may use a single full-bleed photo cropped to 12-px rounded corners.

### Animation
- **Calm and short.** Standard duration ~200ms with `cubic-bezier(0.2, 0, 0, 1)` ("standard" easing).
- Used for: modal fade-in + scale (0.96→1.0), tab/sidebar item color transitions, button press scale (0.98).
- **No bounces.** No spring overshoots. No long entrances. Atlas reads as composed, not playful.

### Hover / press / focus states
- **Hover (filled buttons):** background darkens by ~one step in the scale (e.g. `primary-500 → primary-600`).
- **Hover (outlined / ghost):** background fades to `neutral-25` or the brand-tinted equivalent (e.g. `primary-25`).
- **Press:** transform: scale(0.98) for ≤200ms; no color change required.
- **Focus:** `:focus-visible` only (keyboard, never on mouse click). A 2px `--focus-ring-color` (informative-500) **outline** with 2px offset — not box-shadow, so it never clips against `overflow:hidden` cards/modals. On dark/brand surfaces switch to `--focus-ring-inverse` (white) via `.bg-inverse` / `.filled-dark`. Inputs additionally shift border to informative-500 as an extra affordance, but the outline is the mandatory accessibility ring (WCAG 2.2 SC 2.4.13: ≥2px, ≥3:1 contrast).
- **Disabled:** text/icon to `neutral-500`, surface to `neutral-50`, no opacity changes.

### Borders
- Thin 1px hairlines in `neutral-200` for default; `neutral-100` for "barely-there" dividers; `neutral-300` for stronger separation. Brand outlines for selected/active.

### Shadows & elevation
- Five-step token scale (`xs`→`xl`) using cool-blue `rgba(16,24,40,…)` shadows, NOT pure black. `xs` for chip lift, `sm` for tooltips, `md` for menus, `lg` for popovers, `xl` for modals.
- **No inner shadows** as a default. Inputs use solid 1px borders, not insets.

### Radii
- **8** for buttons/inputs. **12** for cards / image holders inside modals. **16** for modal containers and large surfaces. **20** for badges/pill capsules. **9999** for circular avatars and icon-only chips.
- Atlas favors **rounded-but-not-pill** shapes — buttons are 8 (medium-rounded rectangles), not capsules.

### Cards
- **Surface:** `bg-default` (`#fff`).
- **Border:** `1px solid neutral-200` *or* `shadow-md` — pick one, never both heavy.
- **Radius:** 16 (modal-style) or 12 (compact list cards).
- **Padding:** 16–20.
- **Image inside:** rounded `radius-lg` (12), 16:9 or 16:5 banner aspect.

### Transparency / blur
- Used sparingly. Modal overlays use `--bg-overlay` — `rgb(var(--overlay-tint) / var(--opacity-scrim))`, i.e. tinted eucalipto-black (`2 31 21`) at the `--opacity-scrim` (0.60) wash. No backdrop blur in product canvases. Marketing may use a 6–10 px blur on background photos behind floating cards.

### Layout rules (fixed elements)
- **Sidebar** is fixed at 263 wide on desktop (Figma /Sidebar/Frame-1104220890); collapsed to 64.
- **Top bar** in marketing/admin: 80-px tall, 16-px outer margin, `neutral-25` background, 12-px radius — a "floating" header.
- **Bottom tab bar (mobile):** see /Navegaci-n/Bar.

---

## ICONOGRAPHY

Atlas standardizes on **Phosphor Icons** (`https://phosphoricons.com`). Source link from the Figma's Iconografía page:
`https://www.figma.com/design/XdqMEZypMrscXeVh2fxsIz/Phosphor-Icons`

- **Default weight:** `Light` (1.5px stroke). Some affordances use `Thin` (1px) for delicate UI.
- **Format:** SVG. Phosphor is loaded via CDN here (`@phosphor-icons/web`) so all 9000+ glyphs across `Thin / Light / Regular / Bold / Fill / Duotone` are available.
- **Sizes (token scale):** `2xs 12 / xs 16 / sm 20 / md 24 / lg 32 / xl 48`.
- **Color** flows from `--icon-*` semantic tokens (default = `neutral-900`, subtle = `neutral-700`, brand = `primary-500`, disabled = `neutral-400`, inverse = `white`).

### How to use in HTML

```html
<!-- in <head> -->
<script src="https://code.iconify.design/iconify-icon/2.1.0/iconify-icon.min.js"></script>

<!-- in body -->
<iconify-icon icon="ph:map-pin-light"></iconify-icon>
<iconify-icon icon="ph:airplane-takeoff-light"></iconify-icon>
<iconify-icon icon="ph:bookmark-simple-light"></iconify-icon>
```

### Emoji & Unicode
Not used in the system. Chevrons, bullets, and arrows come from Phosphor (`ph-caret-down`, `ph-caret-right`, `ph-arrow-right`).

### Logo
- `assets/logo-eucalipto-logotype.svg` — full wordmark (preferred default).
- `assets/logo-eucalipto-symbol.svg` — symbol mark only (favicons, tight spaces).
- `assets/logo-taronja-*.svg` — terracota/orange variant for warm photo backgrounds.
- `assets/logo-white-*.svg` — white knockout for dark surfaces.

---

## SUBSTITUTIONS

- The Figma also references DM Sans, Inter, Goldman Sans, IBM Plex Sans, Mail Sans Roman, GitLab Sans, stamPete, Cooper Blk BT — these appear to be Playground/exploration only. **Not part of the Atlas system.** The earlier *Cooper\** display face has been retired; the system uses **Cerebri Heavy** for display.

---

## How to use

1. Link `tokens.css` from any HTML artifact:
   `<link rel="stylesheet" href="tokens.css">`
2. Pull in Phosphor Icons via the CDN snippet above.
3. Compose with the semantic tokens — `var(--bg-default)`, `var(--text-primary)`, `var(--primary-500)`, etc. Avoid hard-coding hex.
4. For full screens / interactive prototypes, see `ui_kits/passporter-app/`.

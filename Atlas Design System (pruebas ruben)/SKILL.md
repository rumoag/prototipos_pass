---
name: passporter-atlas-design
description: Use this skill to generate well-branded interfaces and assets for Passporter (the Atlas design system), either for production or throwaway prototypes/mocks. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for prototyping a calm, photo-led, Spanish-first travel-planning product.
user-invocable: true
---

Read the README.md file within this skill, and explore the other available files.

If creating visual artifacts (slides, mocks, throwaway prototypes, etc), copy assets out and create static HTML files for the user to view. Always link `tokens.css` and load Phosphor Icons via the CDN snippet documented in the README.

If working on production code, you can copy assets and read the rules here to become an expert in designing with this brand.

If the user invokes this skill without any other guidance, ask them what they want to build or design, ask some questions, and act as an expert designer who outputs HTML artifacts _or_ production code, depending on the need.

Key starting points:
- `README.md` — full system docs (voice, foundations, iconography)
- `tokens.css` — drop-in CSS variables and helper classes
- `tokens.json` — raw design tokens
- `assets/` — Passporter logos in eucalipto / taronja / white variants
- `fonts/` — CerebriSans family
- `ui_kits/passporter-app/` — JSX components + interactive demo screen

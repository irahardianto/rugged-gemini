---
name: frontend-design
description: Generates distinctive, production-grade frontend interfaces (React, Vue, HTML/CSS). Bold aesthetics, unique typography, motion. Use for websites, dashboards, posters, or styling tasks.
---

# Frontend Design Skill

Create **distinctive, production-grade interfaces** avoiding generic "AI slop." Follow phased workflow with concrete reference material.

## Phase 1: Design Direction

Before code, commit to aesthetic direction:
- What problem? Who uses it? What's memorable?
- Choose ONE: Brutally minimal · Maximalist · Retro-futuristic · Organic · Luxury · Playful · Editorial · Brutalist · Art deco · Soft/pastel · Industrial · Sci-fi/cyberpunk

**CRITICAL:** Fresh choice every generation. Vary themes, fonts, palettes.

Deliverables: named direction, font pairing (`references/typography.md`), palette (`references/color-palettes.md`), layout (`references/layout-compositions.md`).

## Phase 2: Token System

1. Copy `examples/design-tokens.css` into project
2. Override palette with chosen HSL values
3. Override `--font-display`/`--font-body`
4. Add Google Fonts with preconnect:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="[your @import from typography.md]">
```

Deliverables: tokens in place, all colors/spacing/typography/shadows/motion as CSS vars. Zero hardcoded values.

## Phase 3: Build

Load framework module:

| Target | Module |
|---|---|
| Vue 3 | `frameworks/vue.md` |
| HTML + CSS/JS | `frameworks/html-css.md` |

Principles:
- Components from chosen layout compositions
- Motion patterns (`references/motion-patterns.md`) at: page load, scroll reveals, hover
- Animations via `transform`/`opacity` only — never width/height/top/left
- Every interactive element: hover state AND focus-visible state
- Stagger entrance animations for groups

**Mobile (`references/mobile-responsive.md`):** mobile-first CSS, tap targets ≥44×44px, `100svh` not `100vh`, `env(safe-area-inset-*)` on fixed elements. Test 375/768/1280px.

**PWA (`references/pwa-checklist.md`):** manifest.json, icons (192+512+maskable), service worker, offline fallback. Vite: prefer `vite-plugin-pwa`.

## Phase 4: Polish & Verify

- [ ] Typography: display/body distinct and harmonious?
- [ ] Color: cohesive palette, accents draw eye correctly?
- [ ] Spacing: breathing room (min `--space-20` between major sections)?
- [ ] Motion: alive on load, immediate hover states?
- [ ] Backgrounds: depth/texture/gradient (not flat)?
- [ ] Responsive: great at 375/768/1280px, no horizontal overflow?
- [ ] Touch: all targets ≥44px, mobile nav works?
- [ ] PWA: Lighthouse 100, offline fallback?

Accessibility audit:
```bash
bash .agents/skills/frontend-design/scripts/visual-audit.sh http://localhost:[port]
```
Checks: contrast (WCAG AA 4.5:1), alt text, ARIA, skip link, heading structure, font CLS, `prefers-reduced-motion`. Fix all CRITICAL before complete.

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Gradient Soup | One deliberate gradient for ONE element |
| Font Stack Collapse (all Inter/Roboto) | 2 distinct fonts from typography.md |
| Shadow Boxing (same shadow everywhere) | `--shadow-sm` resting, `--shadow-md` hover |
| Animation Scatter | `--transition-all-interactions` only for interactive |
| Whitespace Desert | Section padding min `--space-16`, hero ≥100svh |
| Button Rainbow | One primary, one secondary, one ghost |
| Flat Backgrounds | Add texture: gradient mesh, noise, pattern |
| Generic Layout | Use compositions from layout-compositions.md |
| Hover Nothing | Every card/button/link needs hover state |
| One-Size Typography | Full type scale: hero→3xl→base |
| Desktop-First CSS | Base=mobile, layer up with min-width |
| Tiny Tap Targets | Min 44×44px interactive area |

## Reference Modules

| Module | When |
|---|---|
| `references/typography.md` | Font choice — 30 pairings |
| `references/color-palettes.md` | Colors — 15 palettes, light+dark |
| `references/motion-patterns.md` | Animations — entrance, hover, scroll |
| `references/layout-compositions.md` | Page structure — 15 compositions |
| `references/mobile-responsive.md` | Mobile-first, touch, viewport |
| `references/pwa-checklist.md` | Manifest, SW, offline, install |
| `frameworks/vue.md` | Vue 3 |
| `frameworks/html-css.md` | HTML/CSS |
| `examples/design-tokens.css` | Token system starter |

## Compliance

Before complete: project-structure, testing-strategy, security-principles (XSS, no innerHTML), accessibility-principles (WCAG AA), visual-audit.sh passes.

Complexity must match vision. Maximalist → elaborate animation. Minimalist → meticulous spacing. Both fail without care. You are capable of extraordinary creative work — references are launching pads, not constraints.

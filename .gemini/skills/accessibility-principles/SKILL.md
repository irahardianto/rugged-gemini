---
name: accessibility-principles
description: >-
  WCAG accessibility: semantic HTML, ARIA, keyboard nav, contrast,
  screen readers. For all user-facing interfaces.
user-invocable: false
---

## Accessibility Principles

Baseline: **WCAG 2.1 Level AA**.

### Semantic HTML
- Use `<nav>`, `<main>`, `<article>`, `<section>`, `<button>`, `<form>` over `<div>`/`<span>`
- One `<h1>` per page, proper hierarchy (no skipping)
- `<label>` associated with inputs (for/id or wrapping)
- `<table>` for data only, never layout

### Keyboard
- All interactive elements Tab-reachable
- Logical focus order
- Visible focus indicators (never `outline: none` without replacement)
- Custom components: arrows for menus, Escape for modals
- No keyboard traps

### ARIA
- First rule: don't use ARIA if native HTML suffices
- `aria-label`/`aria-labelledby` for elements without visible text
- `aria-live` for dynamic updates (toasts, notifications)
- `role` only for custom widgets lacking native semantics
- `aria-expanded`, `aria-controls` for disclosures

### Color & Contrast
- Text: min **4.5:1** (AA normal), **3:1** (AA large)
- Never color alone for meaning (add icons/text/patterns)
- Light + dark themes with proper contrast

### Images & Media
- `alt` text on all `<img>` (`alt=""` for decorative)
- Video captions/transcripts
- Accessible audio controls

### Forms
- Errors associated with inputs (`aria-describedby`)
- Required: visual + programmatic (`required`)
- Validation errors announced to screen readers

### Checklist
- [ ] Keyboard reachable
- [ ] Visible focus indicators
- [ ] Semantic HTML (no div buttons)
- [ ] Heading hierarchy correct
- [ ] Contrast AA (4.5:1)
- [ ] Color not sole indicator
- [ ] Images have alt text
- [ ] Form inputs labeled
- [ ] ARIA correct (native first)

### Related
- Core Design @.gemini/rules/core-design-principles.md
- Security @.gemini/rules/security-principles.md (XSS)
- Testing @.gemini/rules/testing-strategy.md (a11y testing)

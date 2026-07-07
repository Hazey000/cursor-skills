---
name: knowledge-accessibility
description: >-
  Web accessibility principles covering WCAG 2.1 AA, semantic HTML, ARIA,
  keyboard navigation, color contrast, and screen reader compatibility.
  Use when building UI components, reviewing frontend code for accessibility,
  or designing inclusive interfaces.
---

# Accessibility (a11y)

## Purpose

Ensure web interfaces are usable by everyone, including people using assistive technologies.

## POUR Principles (WCAG Foundation)

| Principle | Meaning | Key requirements |
|-----------|---------|-----------------|
| Perceivable | Can users perceive the content? | Alt text, captions, contrast, text resizing |
| Operable | Can users operate the interface? | Keyboard access, no time traps, skip navigation |
| Understandable | Can users understand the content? | Clear language, predictable behaviour, error help |
| Robust | Does it work with assistive tech? | Valid HTML, ARIA where needed, tested with screen readers |

## Semantic HTML First

Use native HTML elements before reaching for ARIA:
- `<button>` not `<div onclick>`
- `<nav>`, `<main>`, `<aside>` for landmarks
- `<h1>`-`<h6>` in order for headings
- `<label>` with `for` attribute for form fields
- `<table>` with proper headers for tabular data

**Rule**: If a native element does the job, don't use ARIA. ARIA is a supplement, not a replacement.

## Keyboard Navigation Essentials

- All interactive elements reachable via Tab
- Logical tab order (follows visual order)
- Visible focus indicators (never `outline: none` without replacement)
- Escape closes modals/popups
- Arrow keys for widget navigation (menus, tabs, sliders)

## Color and Contrast

- Text: minimum 4.5:1 ratio (AA), 3:1 for large text
- UI components: minimum 3:1 ratio
- Never convey information by color alone (use icons, text, patterns too)

## ARIA Essentials

- `aria-label` / `aria-labelledby` for elements without visible text
- `aria-live` regions for dynamic content updates
- `role` only when native semantics don't apply
- `aria-expanded`, `aria-selected`, `aria-checked` for state

## Testing Approach

1. Keyboard-only navigation (unplug mouse)
2. Screen reader testing (VoiceOver, NVDA)
3. Automated tools (axe-core, Lighthouse)
4. Color contrast checker
5. Zoom to 200% — layout should remain usable

## Related Skills

- `frontend/knowledge-component-design` — accessible component patterns
- `reviewers/review-accessibility` — structured accessibility review

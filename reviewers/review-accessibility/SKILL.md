---
name: review-accessibility
description: >-
  Audit for WCAG 2.1 AA compliance covering semantic HTML, ARIA, keyboard
  navigation, color contrast, screen readers, and focus management. Checklist
  organized by POUR principles.
---

# Accessibility Review

Evaluate UI code for WCAG 2.1 Level AA compliance, organized by the POUR principles: Perceivable, Operable, Understandable, Robust.

**Cross-reference**: `frontend/knowledge-accessibility` for WCAG guidelines, ARIA patterns, and semantic HTML.

## When to Use

- Before merging UI changes (new components, layout changes, interactive elements)
- When building forms, modals, navigation, or data tables
- During periodic accessibility audits
- Before launching user-facing features

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | WCAG 2.1 AA failure — content is inaccessible to some users, legal risk — fix before merge |
| **WARNING** | Usability issue for assistive tech users — fix soon |
| **SUGGESTION** | Enhancement that improves experience — implement when practical |

## Review Checklist by POUR Principle

### Perceivable

_Can users perceive all content regardless of ability?_

| Check | WCAG | Severity |
|-------|------|----------|
| Images have descriptive `alt` text (decorative images use `alt=""`) | 1.1.1 | CRITICAL |
| Video/audio has captions or transcripts | 1.2.1 | CRITICAL |
| Color is not the sole means of conveying information | 1.4.1 | CRITICAL |
| Text color contrast ratio ≥ 4.5:1 (normal text), ≥ 3:1 (large text) | 1.4.3 | CRITICAL |
| Non-text contrast (icons, borders, form controls) ≥ 3:1 | 1.4.11 | WARNING |
| Text can be resized to 200% without loss of content | 1.4.4 | WARNING |
| Content reflows at 320px width without horizontal scroll | 1.4.10 | WARNING |
| No information conveyed only through sensory characteristics ("click the red button") | 1.3.3 | WARNING |
| Content order is meaningful in DOM (matches visual order) | 1.3.2 | WARNING |

### Operable

_Can users navigate and interact using keyboard, voice, or assistive tech?_

| Check | WCAG | Severity |
|-------|------|----------|
| All interactive elements are keyboard accessible | 2.1.1 | CRITICAL |
| No keyboard traps (user can always Tab/Escape out) | 2.1.2 | CRITICAL |
| Focus order follows logical reading sequence | 2.4.3 | CRITICAL |
| Focus indicator is visible (not suppressed via `outline: none` without replacement) | 2.4.7 | CRITICAL |
| Skip-to-main-content link provided | 2.4.1 | WARNING |
| Page titles are descriptive | 2.4.2 | WARNING |
| Link text is descriptive (no "click here") | 2.4.4 | WARNING |
| Touch targets ≥ 44x44 CSS pixels | 2.5.5 | WARNING |
| Animations respect `prefers-reduced-motion` | 2.3.3 | WARNING |
| No content that flashes more than 3 times per second | 2.3.1 | CRITICAL |

### Understandable

_Can users understand content and predict interface behavior?_

| Check | WCAG | Severity |
|-------|------|----------|
| `lang` attribute set on `<html>` | 3.1.1 | WARNING |
| Form inputs have associated `<label>` elements | 3.3.2 | CRITICAL |
| Error messages identify the field and describe the error | 3.3.1 | CRITICAL |
| Error suggestions provided when possible | 3.3.3 | WARNING |
| Instructions don't rely solely on visual position | 3.3.2 | WARNING |
| Navigation is consistent across pages | 3.2.3 | WARNING |
| No unexpected context changes on focus or input | 3.2.1/2 | WARNING |

### Robust

_Does the content work with current and future assistive technologies?_

| Check | WCAG | Severity |
|-------|------|----------|
| HTML is valid (proper nesting, no duplicate IDs) | 4.1.1 | WARNING |
| Custom components have correct ARIA roles, states, and properties | 4.1.2 | CRITICAL |
| ARIA attributes match component state (e.g., `aria-expanded` toggles) | 4.1.2 | CRITICAL |
| Status messages use `role="status"` or `aria-live` | 4.1.3 | WARNING |
| Semantic HTML used over generic `<div>`/`<span>` with ARIA | 4.1.2 | WARNING |

## Common Issues

### ARIA Misuse

| Issue | Fix |
|-------|-----|
| `role="button"` on a `<div>` without keyboard handler | Use `<button>` instead, or add `tabindex="0"` + `keydown` handler for Enter/Space |
| `aria-label` on non-interactive element | Move to the interactive element or use `aria-describedby` |
| `aria-hidden="true"` on focusable element | Remove from tab order or remove `aria-hidden` |
| Redundant ARIA (`role="navigation"` on `<nav>`) | Remove the redundant role — native semantics are sufficient |

### Focus Management

| Scenario | Required Behavior |
|----------|-------------------|
| Modal opens | Focus moves to modal, trapped inside, returns to trigger on close |
| Dynamic content added | Announce via `aria-live` region or move focus to new content |
| Route change (SPA) | Focus moves to main content or page heading |
| Item deleted from list | Focus moves to next item or parent container |
| Inline edit activated | Focus moves to input, returns to trigger on save/cancel |

### Form Patterns

| Issue | Fix |
|-------|-----|
| Placeholder as only label | Add visible `<label>` or `aria-label` |
| Required field with no indicator | Add `aria-required="true"` and visible indicator |
| Error shown but not announced | Use `aria-describedby` linking input to error, `aria-invalid="true"` |
| Grouped controls (radio/checkbox) without `<fieldset>`/`<legend>` | Wrap in `<fieldset>` with `<legend>` |

## Output Format

```markdown
## Accessibility Review: [Component/Feature Name]

**Scope**: [What was reviewed]
**Standard**: WCAG 2.1 Level AA
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings by POUR Principle

#### Perceivable
##### [CRITICAL] Missing alt text on product images
**Location**: `src/components/ProductCard.tsx:23`
**WCAG**: 1.1.1 Non-text Content
**Issue**: `<img>` has no `alt` attribute
**Impact**: Screen reader users cannot identify the product
**Fix**: Add descriptive alt: `<img alt={product.name} ... />`

...

### Summary
| Principle | Critical | Warning | Suggestion |
|-----------|----------|---------|------------|
| Perceivable | 1 | 1 | 0 |
| Operable | 0 | 2 | 0 |
| Understandable | 1 | 0 | 1 |
| Robust | 0 | 1 | 0 |
| **Total** | **2** | **4** | **1** |

### Testing Recommendations
- [ ] Test with screen reader (VoiceOver/NVDA)
- [ ] Test keyboard-only navigation
- [ ] Run axe-core automated audit
- [ ] Check color contrast with browser devtools
```

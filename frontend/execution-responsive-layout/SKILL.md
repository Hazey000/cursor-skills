---
name: execution-responsive-layout
description: >-
  Build responsive layouts using modern CSS (Grid, Flexbox, container queries).
  Step-by-step approach to mobile-first design, breakpoint strategy, and fluid
  typography. Use when implementing responsive UIs or converting designs to
  responsive code.
---

# Responsive Layout

## Purpose

Step-by-step procedure for building responsive layouts that work across devices.

## Workflow

### Step 1: Mobile-First Base
Start with the mobile layout as default. No media queries needed for the base.

### Step 2: Define Breakpoints
Choose breakpoints based on content, not devices:
- Content starts to look stretched or awkward → add a breakpoint
- Typical: ~640px (tablet), ~1024px (desktop), ~1280px (wide)

### Step 3: Choose Layout Method

| Scenario | Tool |
|----------|------|
| 1D flow (row or column) | Flexbox |
| 2D grid (rows AND columns) | CSS Grid |
| Component-level responsiveness | Container queries |
| Simple centering/spacing | Flexbox |

### Step 4: Fluid Sizing
- Use `clamp()` for fluid typography: `font-size: clamp(1rem, 2.5vw, 2rem)`
- Use relative units (`rem`, `%`, `vw`) over fixed `px`
- Set `max-width` on content containers

### Step 5: Test
- Resize browser continuously (not just at breakpoints)
- Test with real content (long text, missing images)
- Verify touch targets are ≥44px on mobile

## Checklist

- [ ] Mobile layout works without media queries
- [ ] Breakpoints based on content, not devices
- [ ] No horizontal scrollbar at any width
- [ ] Text readable without zooming on mobile
- [ ] Touch targets ≥44px
- [ ] Images responsive (max-width: 100%)
- [ ] Tested with variable content lengths

## Related Skills

- `frontend/knowledge-accessibility` — responsive design for assistive tech
- `frontend/knowledge-component-design` — responsive component patterns

---
name: execution-form-handling
description: >-
  Design and implement forms with validation, error states, and accessible
  submission flows. Covers validation strategy, error messaging, multi-step
  forms, and optimistic submission. Use when building forms or reviewing form
  implementations.
---

# Form Handling

## Purpose

Step-by-step guidance for building forms that are usable, accessible, and robust.

## Workflow

### Step 1: Design Validation Strategy

| Timing | Use for | Example |
|--------|---------|---------|
| On blur | Individual field validation | Email format |
| On change | Real-time feedback (sparingly) | Password strength |
| On submit | Cross-field validation, server validation | "Passwords must match" |

### Step 2: Error Messaging

- Place errors adjacent to the field (not only at top of form)
- Use `aria-describedby` to associate error with field
- Use `aria-invalid="true"` on invalid fields
- Messages should state what's wrong AND how to fix it
- Use `aria-live="polite"` for dynamically appearing errors

### Step 3: Implement Submission

1. Disable submit button during submission (with loading indicator)
2. Handle server validation errors — map to specific fields
3. On success: clear form or navigate, show confirmation
4. On failure: preserve all entered data, focus first error field

### Step 4: Multi-Step Forms

- Show progress indicator
- Allow back-navigation without data loss
- Validate per-step before advancing
- Save state (consider URL state or session storage)

## Checklist

- [ ] All fields have visible labels (not just placeholders)
- [ ] Required fields marked (asterisk + aria-required)
- [ ] Validation errors associated via aria-describedby
- [ ] Tab order is logical
- [ ] Submit via Enter key works
- [ ] Loading state prevents double submission
- [ ] Server errors mapped to specific fields where possible
- [ ] Form preserves data on submission failure

## Related Skills

- `frontend/knowledge-accessibility` — accessible form patterns
- `frontend/knowledge-state-management` — form state approaches

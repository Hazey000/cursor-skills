---
name: review-ux
description: >-
  Evaluate user experience against Nielsen's heuristics. Covers user flows,
  error states, loading states, empty states, feedback, consistency, and
  information hierarchy.
---

# UX Review

Evaluate UI implementation against Nielsen's 10 usability heuristics, mapped to concrete implementation checks.

**Cross-reference**: `frontend/knowledge-component-design` for component composition and state management patterns.

## When to Use

- Before merging user-facing UI changes
- When building new flows, forms, or interactions
- When redesigning existing features
- During usability review before launch

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Users will be blocked, lose data, or abandon the flow — fix before merge |
| **WARNING** | Users will be confused or frustrated — fix soon |
| **SUGGESTION** | Polish that improves experience — consider for next iteration |

## Review Checklist by Nielsen's Heuristics

### H1: Visibility of System Status

_Users should always know what's happening._

| Check | Severity if violated |
|-------|---------------------|
| Loading states shown for async operations (spinners, skeletons, progress bars) | CRITICAL |
| Progress indication for multi-step processes | WARNING |
| Success feedback after completing actions (save, submit, delete) | WARNING |
| Real-time validation feedback on form inputs | WARNING |
| Network/connection status shown when relevant | SUGGESTION |
| Active state indicated on navigation (current page/tab highlighted) | WARNING |

### H2: Match Between System and Real World

_Use language and concepts familiar to users._

| Check | Severity if violated |
|-------|---------------------|
| Labels and copy use user-facing language, not developer jargon | WARNING |
| Icons are conventional and unambiguous | WARNING |
| Date/time/number formats match user locale expectations | WARNING |
| Workflow order matches user's mental model | WARNING |

### H3: User Control and Freedom

_Users need clear exits and undo._

| Check | Severity if violated |
|-------|---------------------|
| Cancel/back available at every step of multi-step flows | CRITICAL |
| Destructive actions require confirmation | CRITICAL |
| Undo available for reversible actions (delete, archive) | WARNING |
| Modals can be dismissed (close button, Escape key, backdrop click) | WARNING |
| Form data preserved when navigating back | WARNING |
| No dead ends (every state has a clear next action) | CRITICAL |

### H4: Consistency and Standards

_Same action, same result, everywhere._

| Check | Severity if violated |
|-------|---------------------|
| Visual style consistent (spacing, typography, color) across views | WARNING |
| Interaction patterns consistent (save behavior, delete flow) | WARNING |
| Terminology consistent (don't mix "delete" and "remove" for same action) | WARNING |
| Button placement consistent (primary action always in same position) | WARNING |
| Platform conventions followed (form submission, navigation patterns) | SUGGESTION |

### H5: Error Prevention

_Prevent errors before they happen._

| Check | Severity if violated |
|-------|---------------------|
| Input constraints enforced in UI (max length, numeric-only, date pickers) | WARNING |
| Dangerous actions are visually distinct (red for delete, confirmation required) | WARNING |
| Sensible defaults pre-filled where possible | SUGGESTION |
| Inline validation catches errors before submission | WARNING |
| Disabled states with tooltips explaining why (not just grey buttons) | SUGGESTION |

### H6: Recognition Rather Than Recall

_Show options, don't force users to remember._

| Check | Severity if violated |
|-------|---------------------|
| Recently used items / search suggestions available | SUGGESTION |
| Labels visible on form fields (not just placeholders that disappear) | WARNING |
| Context preserved across navigation (filters, selections, scroll position) | WARNING |
| Help text or examples shown where input format isn't obvious | WARNING |

### H7: Flexibility and Efficiency of Use

_Support both novice and expert users._

| Check | Severity if violated |
|-------|---------------------|
| Keyboard shortcuts for frequent actions | SUGGESTION |
| Bulk operations available for lists (select all, batch delete) | SUGGESTION |
| Search/filter for long lists and datasets | WARNING |
| Responsive layout works on mobile and desktop | WARNING |

### H8: Aesthetic and Minimalist Design

_Show only what's needed._

| Check | Severity if violated |
|-------|---------------------|
| Information hierarchy is clear (headings, grouping, whitespace) | WARNING |
| No visual clutter (excessive labels, borders, icons) | SUGGESTION |
| Primary action visually dominant, secondary actions subdued | WARNING |
| Content is scannable (short paragraphs, bullet points, tables) | SUGGESTION |

### H9: Help Users Recognize, Diagnose, and Recover from Errors

_Error messages should be helpful._

| Check | Severity if violated |
|-------|---------------------|
| Error messages explain what went wrong in plain language | CRITICAL |
| Error messages suggest how to fix the problem | WARNING |
| Errors are shown in context (next to the field, not just a banner) | WARNING |
| Form errors don't clear user input | CRITICAL |
| Server errors show a friendly fallback, not raw stack traces | CRITICAL |
| Error states provide a clear recovery path (retry, go back, contact support) | WARNING |

### H10: Help and Documentation

_Provide help where users need it._

| Check | Severity if violated |
|-------|---------------------|
| Complex features have contextual help (tooltips, info icons) | SUGGESTION |
| Onboarding for first-time users of complex flows | SUGGESTION |
| Empty states include guidance on how to get started | WARNING |

## Common State Gaps

Every view should handle these states:

| State | Required | Common miss |
|-------|----------|-------------|
| **Loading** | Skeleton or spinner | Just showing nothing |
| **Empty** | Helpful message + CTA | Blank white space |
| **Error** | Message + retry action | Generic "Something went wrong" |
| **Partial** | Graceful degradation | All-or-nothing rendering |
| **Success** | Confirmation feedback | Silent completion |
| **Offline** | Saved state or warning | Broken UI |

## Output Format

```markdown
## UX Review: [Feature/Flow Name]

**Scope**: [Screens, flows, components reviewed]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings by Heuristic

#### H9: Error Recovery
##### [CRITICAL] Form clears input on validation error
**Location**: `src/components/SignupForm.tsx`
**Issue**: Submitting with invalid email clears all fields
**Impact**: Users lose all entered data and must re-type everything
**Fix**: Preserve form state, show inline error on the email field only

#### H1: System Status
##### [WARNING] No loading state on dashboard
**Location**: `src/pages/Dashboard.tsx`
**Issue**: Dashboard shows blank space while data loads (~2s)
**Fix**: Add skeleton components matching dashboard layout

...

### Summary
| Heuristic | Critical | Warning | Suggestion |
|-----------|----------|---------|------------|
| H1: Visibility | 0 | 1 | 0 |
| H3: Control/Freedom | 1 | 0 | 0 |
| H9: Error Recovery | 1 | 1 | 0 |
| ... | ... | ... | ... |
| **Total** | **N** | **N** | **N** |

### State Coverage
| View | Loading | Empty | Error | Success |
|------|---------|-------|-------|---------|
| Dashboard | ⚠ missing | ✓ | ⚠ generic | ✓ |
| User list | ✓ | ⚠ missing | ✓ | ✓ |

### Recommendations
1. <Prioritized by user impact>
2. ...
```

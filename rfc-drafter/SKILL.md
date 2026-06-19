---
name: rfc-drafter
description: >-
  Draft RFCs (Request for Comments) and technical proposals from conversation
  context. Structures ideas into a formal document with summary, motivation,
  proposal, alternatives, and open questions. Use when the user asks to write
  an RFC, draft a proposal, write a technical design doc, or formalise an idea.
---

# RFC / Proposal Drafter

Turn a technical idea discussed in conversation into a structured RFC or proposal document.

## Core Principle

**Capture the idea, structure the argument, let the user refine.**

Extract everything from conversation context. The user should only need to review and polish, not re-explain.

## Workflow

### Step 1: Extract from Conversation

Scan the current conversation for:
- **The core idea** (what is being proposed)
- **Motivation** (why this matters, what problem it solves)
- **Technical details** (architecture, APIs, data flows, components)
- **Alternatives discussed** (other approaches and why they were rejected or deferred)
- **Open questions** (unresolved points, things that need further investigation)
- **Scope and timeline** (if mentioned)

### Step 2: Confirm Parameters

Use `AskQuestion`:

```
AskQuestion:
  title: "Draft RFC / Proposal"
  questions:
    - id: title
      prompt: "RFC title: '<inferred title>' — looks good?"
      options:
        - "Yes (Recommended)"
        - "No, I'll rephrase"
    - id: audience
      prompt: "Who is the audience?"
      options:
        - "Engineering team (Recommended)"
        - "Wider organisation"
        - "External / open source community"
    - id: format
      prompt: "Output format?"
      options:
        - "Markdown file (Recommended)"
        - "Email-ready (professional prose)"
        - "Display only (copy-paste)"
    - id: depth
      prompt: "Level of detail?"
      options:
        - "Standard (Recommended)"
        - "Lightweight (1-2 pages)"
        - "Comprehensive (full design doc)"
```

### Step 3: Draft the RFC

Use the template matching the requested depth.

#### Standard Template

```markdown
# RFC: [Title]

**Author:** [name]
**Date:** [YYYY-MM-DD]
**Status:** Draft

## Summary

[One paragraph overview of the proposal. A reader should understand the gist
from this section alone.]

## Motivation

[Why is this change needed? What problem does it solve? What happens if we
do nothing? Pull from the conversation context.]

## Proposal

[Detailed technical description of the proposed solution. Include:
- Architecture overview
- Key components and their responsibilities
- API changes or new interfaces
- Data model changes
- Migration strategy (if applicable)]

## Alternatives Considered

### [Alternative 1 name]
[Brief description and why it was rejected or deferred.]

### [Alternative 2 name]
[Brief description and why it was rejected or deferred.]

## Open Questions

- [ ] [Question 1 that needs further investigation]
- [ ] [Question 2 that needs stakeholder input]

## Implementation Plan

| Phase | Description | Estimate |
|-------|-------------|----------|
| 1     | [First milestone] | [timeframe if known] |
| 2     | [Second milestone] | [timeframe if known] |

## References

- [Any links, docs, or prior art mentioned in conversation]
```

#### Lightweight Template

```markdown
# RFC: [Title]

**Author:** [name] | **Date:** [YYYY-MM-DD] | **Status:** Draft

## Summary
[1-2 sentences]

## Motivation
[1-2 paragraphs]

## Proposal
[Core technical approach in 2-4 paragraphs]

## Alternatives Considered
[Brief list with one-line rationale each]

## Open Questions
- [Question 1]
- [Question 2]
```

#### Comprehensive Template

Extends the Standard template with additional sections:

```markdown
## Security Considerations
[Authentication, authorisation, data privacy implications]

## Performance Considerations
[Expected load, latency requirements, scalability concerns]

## Observability
[Logging, metrics, alerting, dashboards needed]

## Rollout Plan
[Feature flags, canary deployment, rollback strategy]

## Success Metrics
[How will we know this was successful? KPIs, acceptance criteria]
```

#### Email Template

```
Subject: [RFC] [Title]

Hi [audience],

I'd like to propose [one-sentence summary].

**Why:** [motivation in 2-3 sentences]

**What:** [proposal in 1-2 paragraphs]

**Alternatives considered:** [brief list]

**Open questions:**
- [Question 1]
- [Question 2]

I've attached/linked the full RFC for detailed review: [link or attachment note]

Looking forward to your feedback.

Best,
[name]
```

### Step 4: Present for Review

Display the full draft and ask:

```
AskQuestion:
  title: "Review RFC"
  questions:
    - id: approve
      prompt: "Review the RFC above. What would you like to do?"
      options:
        - "Looks good, save it (Recommended)"
        - "Adjust the tone or depth"
        - "I want to edit specific sections"
        - "Display only (I'll copy-paste)"
```

### Step 5: Output

Based on user choice:

- **Save**: Write to file. Default path: `docs/rfcs/YYYY-MM-DD-kebab-title.md`. Create the directory if needed.
- **Display only**: Output the final text for copy-paste.
- **Email**: Output the email-formatted version for copy-paste.

Report the file path if saved.

## Tone Guidance by Audience

| Audience | Tone | Characteristics |
|----------|------|-----------------|
| Engineering team | Technical, direct | Use code snippets, architecture details, assume domain knowledge |
| Wider organisation | Clear, accessible | Minimise jargon, focus on impact and outcomes, include a glossary if needed |
| External / open source | Formal, thorough | Full context (no assumed knowledge), clear contribution path, license considerations |

## Important Notes

- Never fabricate technical details not discussed in conversation. If a section needs information the conversation didn't cover, write `[TODO: ...]` as a placeholder.
- Date format is ISO 8601 (`YYYY-MM-DD`).
- The kebab-case filename should match the title (e.g. `2025-06-19-use-event-driven-architecture.md`).
- If the user asks for "a proposal" or "a design doc" without saying "RFC", still use this skill — they're the same deliverable.

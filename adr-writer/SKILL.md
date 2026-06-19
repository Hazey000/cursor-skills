---
name: adr-writer
description: >-
  Capture architecture decisions as ADRs (Architecture Decision Records) using
  the standard Nygard template. Extracts context, decision, and alternatives
  from conversation, auto-numbers the file, and writes it to the project. Use
  when the user asks to record an architecture decision, create an ADR, or
  document a technical decision.
---

# ADR Writer

Capture technical decisions as Architecture Decision Records in the standard Michael Nygard format.

## Core Principle

**Extract from conversation, structure, confirm, write.**

The agent should never ask the user to re-explain a decision that was already discussed. Extract everything possible from conversation context and only confirm.

## Workflow

### Step 1: Extract from Conversation

Scan the current conversation for:
- **What was decided** (the core technical choice)
- **Why it was decided** (motivation, constraints, requirements)
- **What alternatives were considered** (other options discussed and why they were rejected)
- **What the consequences are** (trade-offs, follow-up work, risks)

### Step 2: Confirm Details

Use `AskQuestion` to confirm:

```
AskQuestion:
  title: "Create ADR"
  questions:
    - id: title
      prompt: "ADR title: '<inferred title>' — looks good?"
      options:
        - "Yes (Recommended)"
        - "No, I'll rephrase"
    - id: status
      prompt: "Status?"
      options:
        - "Accepted (Recommended)"
        - "Proposed"
        - "Deprecated"
        - "Superseded"
    - id: directory
      prompt: "Where should ADRs be saved?"
      options:
        - "docs/adr/ (Recommended)"
        - "docs/decisions/"
        - "adr/"
```

### Step 3: Auto-Number

Determine the next ADR number:

1. Use `Glob` to list existing files in the target directory matching `*.md`
2. Parse the highest existing number (e.g. `0005-use-postgres.md` -> next is `0006`)
3. If no ADRs exist yet, start at `0001`

Format: `NNNN-kebab-case-title.md` (e.g. `0003-use-redis-for-caching.md`)

### Step 4: Draft the ADR

Use the Nygard template:

```markdown
# N. Title

Date: YYYY-MM-DD

## Status

[Proposed | Accepted | Deprecated | Superseded by [ADR-NNNN](NNNN-title.md)]

## Context

[What is the issue that we're seeing that is motivating this decision or change?
Pull this from the conversation — the problem being solved, the constraints,
the requirements that led to this decision point.]

## Decision

[What is the change that we're proposing and/or doing?
State the decision clearly and concisely.]

## Consequences

[What becomes easier or more difficult to do because of this change?
Include both positive and negative consequences. Mention any follow-up
work or risks.]
```

**Writing guidelines:**
- **Context** should read as background for someone who wasn't in the conversation. Write in third person, present tense.
- **Decision** should be a clear, declarative statement. Start with "We will..." or "We have decided to...".
- **Consequences** should list concrete trade-offs, not vague statements. Both positive ("This simplifies X") and negative ("This means we cannot easily Y").

### Step 5: Present for Review

Display the full ADR draft and ask:

```
AskQuestion:
  title: "Review ADR"
  questions:
    - id: approve
      prompt: "Review the ADR above. Ready to save?"
      options:
        - "Save it (Recommended)"
        - "I want to edit it first"
        - "Cancel"
```

### Step 6: Write the File

On approval:

1. Create the target directory if it doesn't exist (use `Shell` with `mkdir -p`)
2. Write the ADR file using the `Write` tool
3. Report the file path back to the user

## Handling Edge Cases

- **No conversation context**: If the user invokes the skill without a prior decision discussion, ask them to describe the decision, alternatives, and rationale.
- **Superseding an existing ADR**: If the user mentions this replaces a previous decision, set the new ADR status to "Accepted" and note which ADR it supersedes. Remind the user to update the old ADR's status to "Superseded by [ADR-NNNN]".
- **Multiple decisions in one conversation**: If the conversation contains several distinct decisions, ask which one to record, or offer to create multiple ADRs sequentially.

## Important Notes

- Always use 4-digit zero-padded numbers (`0001`, not `1`).
- The kebab-case title in the filename should be short (3-6 words max).
- Date format is ISO 8601 (`YYYY-MM-DD`).
- Never overwrite an existing ADR — always create a new numbered file.
- If the target directory doesn't exist, create it automatically.

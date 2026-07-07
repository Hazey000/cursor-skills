---
name: execution-user-story-mapping
description: >-
  Map user journeys into development slices using story mapping. Step-by-step
  procedure for creating a story map, identifying releases, and ensuring
  end-to-end user value in each slice. Use when planning features, defining
  sprints, or organizing backlogs.
---

# User Story Mapping

## Purpose

Transform user goals into ordered development slices that deliver end-to-end value.

## Workflow

### Step 1: Identify the User and Goal
- Who is the user persona?
- What is their high-level goal? (the "backbone" of the map)

### Step 2: Map the Journey (Left to Right)
List the high-level activities in the order the user performs them:
```
[Sign Up] → [Configure] → [Create Content] → [Publish] → [Analyze]
```

### Step 3: Break Activities into Tasks (Top to Bottom)
Under each activity, list the specific tasks, ordered by priority:
```
[Sign Up]        [Create Content]
- Email/password  - New post
- OAuth login     - Add images
- Invite flow     - Embed video
                  - Draft/schedule
```

### Step 4: Draw Release Lines (Horizontal Slices)

Draw horizontal lines to define releases:
- **Release 1 (MVP)**: Top row of each activity — minimum viable journey
- **Release 2**: Next priority row — enhanced journey
- **Release 3+**: Remaining enhancements

**Key insight**: Each release must tell a complete (if basic) story from left to right.

### Step 5: Validate Each Slice

For each release slice, verify:
- [ ] User can complete their goal end-to-end
- [ ] No dead ends (every step has a next step or completion)
- [ ] Provides testable value (can measure success)

## Anti-Patterns

- **Vertical slicing** (one activity fully complete, others empty): User can't complete journey
- **Technical slices** ("build database layer"): No user-facing value
- **Too thin MVP**: Unusable even if technically end-to-end

## Related Skills

- `product/knowledge-mvp-strategy` — what goes in each slice
- `product/knowledge-prioritization` — ordering within slices
- `product/knowledge-product-thinking` — user-centric development

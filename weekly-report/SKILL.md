---
name: weekly-report
description: >-
  Generate a weekly status report by aggregating Git commits, JIRA tickets,
  GitHub issues, and pull requests. Categorises work into completed, in-progress,
  and blockers. Use when the user asks for a status report, weekly summary,
  standup notes, or wants to summarise what they worked on.
---

# Weekly Status Report

Generate a formatted weekly status report via GitKraken MCP tools.

## Core Principle

**Gather, categorise, format, review.**

Pull data from all available sources, organise it into a coherent narrative, and let the user review before finalising.

## Workflow

### Step 1: Confirm Parameters

Use `AskQuestion` to confirm:

```
AskQuestion:
  title: "Weekly Status Report"
  questions:
    - id: period
      prompt: "Reporting period?"
      options:
        - "Last 7 days (Recommended)"
        - "Last 5 days (working week)"
        - "Custom range"
    - id: format
      prompt: "Output format?"
      options:
        - "Markdown (Recommended)"
        - "Slack-ready (concise, emoji headers)"
        - "Email (professional prose)"
    - id: sources
      prompt: "Which data sources to include?"
      allow_multiple: true
      options:
        - "Git commits (Recommended)"
        - "JIRA tickets (Recommended)"
        - "GitHub issues"
        - "Pull requests (Recommended)"
```

### Step 2: Gather Data

Pull data from all selected sources **in parallel**:

**Git commits:**
```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: git_log_or_diff
  arguments:
    directory: "<current workspace>"
    action: "log"
    since: "7 days ago"           # adjust to reporting period
```

**JIRA tickets:**
```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: issues_assigned_to_me
  arguments:
    provider: "jira"
```

**GitHub issues:**
```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: issues_assigned_to_me
  arguments:
    provider: "github"
```

**Pull requests (open):**
```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: pull_request_assigned_to_me
  arguments:
    provider: "github"
```

**Pull requests (recently closed/merged):**
```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: pull_request_assigned_to_me
  arguments:
    provider: "github"
    is_closed: true
```

### Step 3: Categorise

Organise all gathered data into these categories:

| Category | What goes here |
|----------|---------------|
| **Completed** | Merged PRs, closed tickets, resolved issues |
| **In Progress** | Open PRs, active tickets, in-review items |
| **Blockers** | Any items mentioned as blocked in ticket descriptions or conversation |
| **Up Next** | Open tickets not yet started, upcoming work inferred from backlog |

Deduplicate across sources — a merged PR that references a JIRA ticket should appear once, not twice.

### Step 4: Draft the Report

Use the format template matching the user's selection.

#### Markdown Template

```markdown
# Weekly Status Report
**Period:** [start date] – [end date]
**Author:** [name]

## Completed
- [PR #123](url) — Brief description (closes TICKET-456)
- TICKET-789 — Brief description

## In Progress
- [PR #124](url) — Brief description, currently in review
- TICKET-012 — Brief description

## Blockers
- [Description of blocker and what it's blocking]

## Up Next
- TICKET-345 — Brief description
- [Planned work inferred from open tickets]
```

#### Slack Template

```
*Weekly Status Report* — [start date] – [end date]

*Completed*
• [item] — brief description
• [item] — brief description

*In Progress*
• [item] — brief description

*Blockers*
• [description]

*Up Next*
• [item]
```

#### Email Template

```
Subject: Weekly Status — [start date] – [end date]

Hi team,

Here's a summary of my work this past week.

**Completed:** [prose summary of merged PRs and closed tickets]

**In Progress:** [prose summary of active work]

**Blockers:** [any blockers, or "None at this time."]

**Next Week:** [planned work]

Best,
[name]
```

### Step 5: Present for Review

Display the draft report and ask for approval:

```
AskQuestion:
  title: "Review Status Report"
  questions:
    - id: approve
      prompt: "Here's your weekly status report. How does it look?"
      options:
        - "Looks good, finalise it (Recommended)"
        - "I want to edit it"
        - "Regenerate with different format"
```

### Step 6: Output

On approval, output the final report. If the user requested markdown, offer to save it to a file (e.g. `reports/weekly-YYYY-MM-DD.md`).

## Important Notes

- The `git_log_or_diff` tool requires a `directory` argument — use the current workspace path.
- The `since` field accepts relative dates like `"7 days ago"`, `"5 days ago"`, `"2 weeks ago"`.
- `pull_request_assigned_to_me` with `is_closed: true` returns recently closed/merged PRs.
- Cross-reference PR titles and ticket keys to deduplicate (e.g. a PR titled "PROJ-123: Fix bug" maps to ticket PROJ-123).
- If a data source returns no results, omit that section rather than showing an empty category.
- Adapt the author name from the git log or JIRA assignee field.

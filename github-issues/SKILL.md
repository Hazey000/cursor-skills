---
name: github-issues
description: >-
  Create GitHub issues on any repo from a context document or conversation.
  Parses raw input, formats and improves issue titles and descriptions, infers
  labels and assignees, fills in missing fields, presents a review table, and
  creates issues on approval. Use when the user asks to create GitHub issues,
  file issues from a doc, bulk-create issues, or log work on a GitHub repo.
---

# GitHub Issue Creation

Create GitHub issues via the GitKraken MCP `issues_create` tool with `provider: "github"`.

## Core Principle

**Parse, improve, review, create.**

1. Read the user's context doc or conversation and extract raw issue descriptions
2. Improve each issue: sharpen titles, structure descriptions, infer labels and assignees
3. Present a review table — never create without approval
4. On approval, create issues and report back with URLs

## Workflow

### Step 1: Identify the Target Repo

Use `AskQuestion` to confirm:
- **Repository owner** (GitHub org or username)
- **Repository name**

If the user has already specified these, confirm with a recommended default.

### Step 2: Parse the Context Document

Read the provided file or conversation context and extract individual issues. Look for:
- Numbered lists, bullet points, or headings that delineate separate items
- Bug reports, feature requests, tasks, or improvements
- Any mentioned priorities, labels, or assignees

For each extracted item, capture:
- Raw title or summary
- Any description/detail text
- Any labels or categories mentioned
- Any people mentioned

### Step 3: Improve and Enrich Each Issue

For every extracted issue, apply these improvements:

**Title**: Rewrite as a clear, actionable issue title.

| Raw input | Improved title |
|-----------|---------------|
| "the login page is slow" | Improve login page load performance |
| "add dark mode" | Add dark mode theme toggle |
| "crash when uploading large files" | Fix crash on large file upload |
| "need API docs" | Add API documentation |

**Description**: Structure using the appropriate template (see below).

**Labels**: Infer from context:

| Signal | Suggested label |
|--------|----------------|
| Something is broken, crash, error, regression | `bug` |
| New capability, "add", "implement", "support" | `enhancement` |
| Documentation, README, guides | `documentation` |
| Performance, slow, optimise | `performance` |
| Security, vulnerability, auth | `security` |
| Refactor, clean up, technical debt | `refactor` |
| Question, unclear behaviour | `question` |

**Assignees**: Include GitHub usernames if mentioned. Otherwise leave unassigned.

### Step 4: Present for Review

Present **all** issues in a numbered table before creating any. Include columns:

| # | Title | Labels | Assignee | Summary |
|---|-------|--------|----------|---------|

Then ask for approval:

```
AskQuestion:
  title: "Review GitHub Issues"
  questions:
    - id: approve
      prompt: "I've prepared N issues for owner/repo. Review the table above — ready to create?"
      options:
        - "Create all (Recommended)"
        - "Let me edit some first"
        - "Cancel"
```

If the user selects "Let me edit some first", ask which issues to change and re-present.

### Step 5: Create Issues

On approval, create each issue sequentially:

```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: issues_create
  arguments:
    provider: "github"
    repository_organization: "<OWNER>"
    repository_name: "<REPO>"
    title: "<TITLE>"
    body: "<DESCRIPTION in markdown>"
    labels: ["<label1>", "<label2>"]    # omit if none
    assignees: ["<username>"]           # omit if unassigned
```

### Step 6: Report Results

After all issues are created, present a summary table:

| # | Title | Issue | URL |
|---|-------|-------|-----|

Include the issue number and URL for each. Flag any that failed.

## Description Templates

### Feature / Enhancement

```markdown
## Summary
[1-2 sentence overview of the feature]

## Motivation
[Why this is needed — pull from context doc]

## Proposed Solution
[How it could be implemented, if context provides this]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Additional Context
[Any extra detail, links, or references from the source doc]
```

### Bug

```markdown
## Summary
[What is broken]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]

## Expected Behaviour
[What should happen]

## Actual Behaviour
[What actually happens]

## Environment
[OS, browser, version, etc. if mentioned]

## Additional Context
[Logs, screenshots, related issues]
```

### Task / Chore

```markdown
## Summary
[What needs to be done]

## Details
[Any specifics from the context doc]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
```

## Single Issue Mode

When the user describes just one issue (not a doc), follow the same flow but simplified:

1. Infer title, description, and labels from conversation
2. Present a quick confirmation via `AskQuestion` (title, labels, assignee)
3. Create on approval

## Important Notes

- `repository_organization` is the GitHub **owner** (org or username), `repository_name` is the **repo name**.
- Labels must already exist on the repo — the MCP tool applies existing labels by name. If a label doesn't exist, it will be silently ignored. Warn the user if you're suggesting uncommon labels.
- If the MCP call fails with an auth error, tell the user to check their GitKraken GitHub integration in the GitLens extension settings.
- Always present created issue details (number + URL) back to the user.
- Never create issues without explicit user approval.

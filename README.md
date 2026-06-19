# Cursor Agent Skills

A collection of reusable [Cursor](https://cursor.com) agent skills.

## Skills

### JIRA Ticket Creation

**Path:** `jira-tickets/SKILL.md`

Create JIRA tickets at any hierarchy level (Epic, Story, Task, Sub-task, Bug) via the GitKraken MCP integration. The skill infers as much as possible from conversation context and only asks for what it can't determine.

**Requirements:**
- [GitLens extension](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) with a connected JIRA integration
- GitKraken MCP server enabled in Cursor

**Usage:**
1. Copy `jira-tickets/SKILL.md` into your project's `.cursor/skills/jira-tickets/` directory
2. Ask the agent to create a JIRA ticket — it will pick up the skill automatically

### GitHub Issue Creation

**Path:** `github-issues/SKILL.md`

Create GitHub issues on any repo from a context document or conversation. The skill parses raw input, improves titles and descriptions, infers labels, presents a review table for approval, and then creates the issues.

**Requirements:**
- [GitLens extension](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) with a connected GitHub integration
- GitKraken MCP server enabled in Cursor

**Usage:**
1. Copy `github-issues/SKILL.md` into your project's `.cursor/skills/github-issues/` directory
2. Provide a context doc or describe the issues you want to create
3. The agent will parse, improve, and present them for review before creating

### Weekly Status Report

**Path:** `weekly-report/SKILL.md`

Generate a weekly status report by aggregating Git commits, JIRA tickets, GitHub issues, and pull requests. Categorises work into completed, in-progress, and blockers, then formats as markdown, Slack-ready, or email.

**Requirements:**
- [GitLens extension](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) with connected JIRA and/or GitHub integrations
- GitKraken MCP server enabled in Cursor

**Usage:**
1. Copy `weekly-report/SKILL.md` into your project's `.cursor/skills/weekly-report/` directory
2. Ask the agent for a weekly status report or standup summary

### ADR Writer

**Path:** `adr-writer/SKILL.md`

Capture architecture decisions as ADRs (Architecture Decision Records) using the standard Nygard template. Extracts context, decision, and alternatives from conversation, auto-numbers the file, and writes it to the project.

**Requirements:**
- No external integrations required — works with the filesystem only

**Usage:**
1. Copy `adr-writer/SKILL.md` into your project's `.cursor/skills/adr-writer/` directory
2. Discuss a technical decision in conversation, then ask the agent to record it as an ADR

### RFC / Proposal Drafter

**Path:** `rfc-drafter/SKILL.md`

Draft RFCs and technical proposals from conversation context. Structures ideas into a formal document with summary, motivation, proposal, alternatives, and open questions. Supports multiple output formats and depth levels.

**Requirements:**
- No external integrations required — works with the filesystem only

**Usage:**
1. Copy `rfc-drafter/SKILL.md` into your project's `.cursor/skills/rfc-drafter/` directory
2. Discuss a technical idea in conversation, then ask the agent to draft an RFC or proposal

## License

MIT

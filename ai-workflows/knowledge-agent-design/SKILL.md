---
name: knowledge-agent-design
description: >-
  AI agent architecture patterns covering tool use, planning, memory, and
  multi-agent coordination. Decision framework for agent design choices.
  Use when designing AI agent systems, choosing agent architectures, or
  reviewing agent implementations.
---

# Agent Design

## Purpose

Decision frameworks for designing effective AI agent systems.

## Agent Architecture Patterns

| Pattern | Complexity | Best for |
|---------|-----------|----------|
| Single-turn tool use | Low | Straightforward tasks with clear tool mapping |
| ReAct (Reason + Act) | Medium | Multi-step tasks requiring planning |
| Plan-then-execute | Medium | Tasks where the full plan can be determined upfront |
| Reflexion/self-critique | High | Tasks requiring quality iteration |
| Multi-agent collaboration | High | Complex tasks benefiting from specialization |

## Tool Design Principles

- **Atomic tools**: Each tool does one thing well
- **Clear interfaces**: Typed inputs/outputs, explicit constraints
- **Idempotent where possible**: Safe to retry on failure
- **Observable**: Return enough context for the agent to verify success
- **Fail informatively**: Error messages that guide correction

## Memory Patterns

| Type | Duration | Use for |
|------|----------|---------|
| Context window | Single conversation | Working state |
| Conversation summary | Session | Long interactions |
| Retrieval (RAG) | Persistent | Large knowledge bases |
| Structured memory | Persistent | Facts, preferences, state |

## Agent Failure Modes

| Failure | Cause | Mitigation |
|---------|-------|------------|
| Infinite loops | No termination condition | Max iterations, loop detection |
| Tool misuse | Ambiguous tool descriptions | Better descriptions, examples |
| Hallucinated actions | No validation of outputs | Output validation, guardrails |
| Context overflow | Too much history | Summarization, relevance filtering |
| Cascading errors | Unchecked failure propagation | Error boundaries, rollback |

## Design Checklist

- [ ] Clear termination conditions
- [ ] Maximum iteration limits
- [ ] Error recovery strategy
- [ ] Output validation/guardrails
- [ ] Fallback to human when uncertain
- [ ] Observability (log decisions, not just actions)

## Related Skills

- `ai-workflows/knowledge-prompt-engineering` — prompting agents effectively
- `ai-workflows/knowledge-guardrails` — safety and validation
- `ai-workflows/execution-rag-pipeline` — retrieval for agent knowledge

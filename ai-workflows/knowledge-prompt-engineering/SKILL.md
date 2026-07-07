---
name: knowledge-prompt-engineering
description: >-
  Prompt engineering patterns covering system prompts, few-shot examples,
  chain-of-thought, structured output, and prompt testing. Use when writing
  prompts for AI systems, designing agent instructions, or optimizing LLM
  interactions.
---

# Prompt Engineering

## Purpose

Design prompts that produce reliable, high-quality outputs from language models.

## Core Patterns

| Pattern | When to use | Effect |
|---------|-------------|--------|
| System prompt | Always | Set role, constraints, format |
| Few-shot examples | Output format matters | Demonstrates expected behavior |
| Chain-of-thought | Complex reasoning | Improves accuracy on multi-step problems |
| Structured output | Parsing needed | JSON/XML schema enforcement |
| Step-by-step decomposition | Complex tasks | Breaks into manageable sub-tasks |

## System Prompt Design

Structure:
1. **Role**: Who the model is (expertise, perspective)
2. **Task**: What it should do
3. **Constraints**: What it must not do
4. **Format**: How to structure output
5. **Examples**: Representative input/output pairs

## Prompt Reliability Techniques

| Technique | Purpose |
|-----------|---------|
| Be specific | Reduce ambiguity in interpretation |
| Provide format template | Ensure parseable output |
| Specify edge cases | "If X is missing, do Y" |
| Add constraints | "Never include PII", "Maximum 3 sentences" |
| Request self-verification | "Check your answer against..." |

## Common Failure Modes and Fixes

| Problem | Fix |
|---------|-----|
| Inconsistent format | Add explicit format template + few-shot |
| Hallucinated data | Request citations, add "if unsure, say so" |
| Too verbose | "Be concise. Maximum N sentences." |
| Ignores constraints | Move constraints to end (recency bias) or repeat |
| Poor reasoning | Add chain-of-thought, decompose problem |

## Prompt Testing

- Test with diverse inputs (not just happy path)
- Check edge cases (empty input, very long input, ambiguous input)
- Measure consistency (same input multiple times)
- Track prompt versions (like code versions)

## Anti-Patterns

- **Mega-prompts**: Thousands of tokens of instructions (diminishing returns)
- **Conflicting instructions**: Contradictory rules that confuse the model
- **Assumed context**: Instructions that depend on information not provided
- **No examples for complex formats**: Expecting specific structure without showing it

## Related Skills

- `ai-workflows/knowledge-agent-design` — prompts for agent systems
- `ai-workflows/knowledge-guardrails` — safety constraints in prompts

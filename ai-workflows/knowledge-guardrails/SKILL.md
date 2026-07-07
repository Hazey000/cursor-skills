---
name: knowledge-guardrails
description: >-
  AI safety guardrails covering input validation, output filtering, fallback
  strategies, and monitoring for AI systems. Use when designing safety
  mechanisms for AI-powered features or reviewing AI system reliability.
---

# Guardrails

## Purpose

Design safety mechanisms that prevent AI systems from producing harmful or incorrect outputs.

## Guardrail Layers

| Layer | What it catches | Implementation |
|-------|----------------|----------------|
| Input validation | Malformed, malicious, or out-of-scope requests | Schema validation, content filtering |
| Prompt injection defense | Attempts to override instructions | Input sanitization, instruction hierarchy |
| Output validation | Incorrect format, hallucinated data, unsafe content | Schema validation, fact-checking, content filter |
| Rate limiting | Abuse, cost explosion | Token/request limits per user |
| Fallback | Complete failure | Graceful degradation to non-AI path |

## Input Guardrails

- Validate input length (prevent context stuffing)
- Filter known injection patterns
- Classify intent (reject out-of-scope requests)
- Sanitize user input before injecting into prompts

## Output Guardrails

- Parse output against expected schema (reject malformed)
- Check for PII leakage before returning to user
- Validate factual claims against known data sources
- Content safety filtering (toxicity, bias)
- Confidence thresholds (if model reports low confidence, fall back)

## Fallback Strategy

When AI fails:
1. Return cached/default response if safe
2. Fall back to rule-based system
3. Escalate to human
4. Return honest "I don't know" with next steps

**Never**: Silently return potentially wrong answers as if they're correct.

## Monitoring

- Track fallback rate (increasing = prompt/model degradation)
- Monitor output quality over time (drift detection)
- Log guardrail trigger reasons (identify patterns)
- Alert on unusual request patterns (potential abuse)

## Related Skills

- `ai-workflows/knowledge-agent-design` — guardrails in agent systems
- `ai-workflows/knowledge-prompt-engineering` — defensive prompt design
- `security/knowledge-secure-by-design` — AI security context

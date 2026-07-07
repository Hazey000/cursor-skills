---
name: execution-api-docs
description: >-
  Write effective API reference documentation. Covers structure, examples,
  error documentation, and maintaining accuracy. Use when writing or reviewing
  API documentation.
---

# API Documentation

## Purpose

Write API documentation that enables developers to integrate successfully without support.

## Structure

1. **Overview**: What the API does, who it's for (2-3 sentences)
2. **Authentication**: How to get and use credentials
3. **Quickstart**: Working example in <5 minutes
4. **Endpoints reference**: Complete endpoint documentation
5. **Error reference**: All error codes and recovery guidance
6. **Rate limits**: Limits and what happens when exceeded
7. **Changelog**: Breaking changes with migration guidance

## Endpoint Documentation Template

For each endpoint:
- HTTP method + path
- One-sentence description
- Request parameters (path, query, body) with types and constraints
- Response shape with field descriptions
- At least one complete request/response example
- Error cases specific to this endpoint

## Quality Markers

- [ ] Every example is copy-pasteable and works
- [ ] All required vs optional params clearly marked
- [ ] Error responses documented with recovery action
- [ ] Authentication covered before any endpoint
- [ ] Common use cases have dedicated examples

## Related Skills

- `api/execution-api-documentation` — detailed API doc writing process
- `api/knowledge-rest-principles` — API design that documents well
- `documentation/knowledge-technical-writing` — general writing principles

---
name: execution-api-documentation
description: >-
  How to write effective API documentation. Covers structure (overview, auth,
  quickstart, endpoint reference, errors, rate limits, changelog), OpenAPI best
  practices, example design, and common anti-patterns. Use when writing or
  reviewing API documentation.
---

# API Documentation Procedure

How to produce documentation that developers actually use.

## Documentation Structure

Every API doc should have these sections, in this order. Developers scan top-down — put the most important things first.

### 1. Overview

One paragraph. What this API does, who it's for, and what you can build with it.

```markdown
## Overview

The Orders API lets you create, manage, and fulfill customer orders
programmatically. Use it to integrate order management into your
application, automate fulfillment workflows, or build custom dashboards.
```

**Don't:** Repeat the company mission statement, explain what REST is, or include marketing copy.

### 2. Authentication

How to get credentials and how to use them. This is the first thing a developer needs to get started.

```markdown
## Authentication

All requests require a Bearer token in the `Authorization` header:

    Authorization: Bearer sk_live_abc123

**Getting a token:** Go to [Dashboard → API Keys](https://example.com/settings/keys)
to create a key. Use `sk_test_` keys for development and `sk_live_` keys for production.

**Scopes:** Keys can be scoped to specific permissions:
| Scope | Access |
|-------|--------|
| `orders:read` | List and retrieve orders |
| `orders:write` | Create and modify orders |
| `orders:delete` | Delete orders |
```

### 3. Quickstart

A complete, copy-paste working example that a developer can run in under 2 minutes. This is the single most important section.

```markdown
## Quickstart

Create your first order in 3 steps:

**1. Install the SDK** (or use curl)

    npm install @example/orders-sdk

**2. Create an order**

    import { OrdersClient } from '@example/orders-sdk';

    const client = new OrdersClient({ apiKey: 'sk_test_abc123' });

    const order = await client.orders.create({
      customer_id: 'cust_123',
      items: [{ product_id: 'prod_456', quantity: 1 }]
    });

    console.log(order.id); // ord_789

**3. Check the order status**

    const retrieved = await client.orders.get('ord_789');
    console.log(retrieved.status); // 'pending'
```

**Quickstart rules:**
- Must be runnable — no pseudocode, no `// your code here`
- Use test/sandbox credentials in examples
- Show the response so developers know it worked
- Under 20 lines of actual code
- Include both SDK and raw HTTP (curl) examples

### 4. Endpoint Reference

Systematic documentation for every endpoint. Consistency matters more than prose quality.

**Per-endpoint template:**

```markdown
### Create Order

    POST /v1/orders

Creates a new order.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customer_id` | string | Yes | The customer's ID |
| `items` | array | Yes | Line items (min 1) |
| `items[].product_id` | string | Yes | Product ID |
| `items[].quantity` | integer | Yes | Quantity (min 1) |
| `shipping_address` | object | No | Defaults to customer's saved address |
| `idempotency_key` | string | No | Prevents duplicate orders |

**Response** `201 Created`

    {
      "id": "ord_789",
      "status": "pending",
      "customer_id": "cust_123",
      "items": [
        {
          "product_id": "prod_456",
          "quantity": 1,
          "unit_price": 29.99
        }
      ],
      "total": 29.99,
      "created_at": "2025-01-15T10:30:00Z"
    }

**Errors**

| Status | Code | When |
|--------|------|------|
| 422 | `VALIDATION_ERROR` | Missing required fields or invalid values |
| 409 | `DUPLICATE_ORDER` | Idempotency key already used |
| 401 | `UNAUTHORIZED` | Invalid or missing API key |
```

**Reference rules:**
- Show every field, including nested objects
- Include realistic example values (not "string", "123")
- Document every possible error for this specific endpoint
- Specify which fields are required vs optional
- Note default values for optional fields

### 5. Error Reference

A single page listing all error codes, their meaning, and how to handle them.

```markdown
## Errors

All errors follow this format:

    {
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "Human-readable description",
        "details": [ ... ],
        "request_id": "req_abc123"
      }
    }

Always include `request_id` when contacting support.

### Error Codes

| Code | HTTP Status | Meaning | How to Fix |
|------|-------------|---------|------------|
| `VALIDATION_ERROR` | 422 | Invalid request body | Check `details` array for specific fields |
| `NOT_FOUND` | 404 | Resource doesn't exist | Verify the ID |
| `UNAUTHORIZED` | 401 | Invalid credentials | Check API key |
| `FORBIDDEN` | 403 | Insufficient permissions | Check key scopes |
| `RATE_LIMITED` | 429 | Too many requests | Back off and retry after `Retry-After` |
| `CONFLICT` | 409 | State conflict | Read current state and retry |
| `INTERNAL_ERROR` | 500 | Server error | Retry with backoff; contact support if persistent |
```

**The "How to Fix" column is critical.** It's the difference between docs that solve problems and docs that describe problems.

### 6. Rate Limits

```markdown
## Rate Limits

| Plan | Limit | Window |
|------|-------|--------|
| Free | 100 requests | per minute |
| Pro | 1,000 requests | per minute |
| Enterprise | Custom | Custom |

Rate limit headers are included in every response:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Max requests per window |
| `X-RateLimit-Remaining` | Requests remaining |
| `X-RateLimit-Reset` | Unix timestamp when window resets |

When rate limited, you'll receive a `429` response. Use exponential
backoff with jitter, respecting the `Retry-After` header.
```

### 7. Changelog

```markdown
## Changelog

### 2025-03-01 — v1.2.0
- Added `shipping_address` field to order creation
- Added `GET /v1/orders/{id}/tracking` endpoint

### 2025-02-01 — v1.1.0
- Added cursor-based pagination to `GET /v1/orders`
- **Deprecation:** Offset pagination will be removed on 2025-08-01

### 2025-01-01 — v1.0.0
- Initial release
```

**Changelog rules:**
- Date-descending order
- Separate additions, changes, deprecations, and removals
- Link to migration guides for breaking changes
- Call out deprecations clearly

## OpenAPI / Swagger Best Practices

| Practice | Why |
|---|---|
| Add `description` to every path, operation, parameter, and schema | Drives generated docs quality |
| Use `example` on every schema property | Powers "Try it" features and mock servers |
| Use `$ref` for all reusable schemas | Single source of truth, smaller spec |
| Set `operationId` on every endpoint | Drives SDK method names in code generators |
| Use `tags` to group related endpoints | Organizes generated docs into sections |
| Add `externalDocs` for deep-dive guides | Links spec to tutorial content |
| Version the spec file itself in source control | Track spec changes alongside code changes |

### Spec hygiene:

```yaml
paths:
  /orders/{id}:
    get:
      tags: [Orders]
      summary: Retrieve an order         # Short (< 10 words)
      description: |                      # Detailed explanation
        Returns a single order by ID. Includes all line items,
        shipping details, and current status.
      operationId: getOrder               # Unique, camelCase
      parameters:
        - name: id
          in: path
          required: true
          description: The order's unique identifier
          schema:
            type: string
            example: ord_789              # Realistic example
```

## Writing Good Examples

Examples are the most-read part of API docs. Invest here.

| Principle | Good | Bad |
|---|---|---|
| Realistic data | `"name": "Alice Johnson"` | `"name": "string"` |
| Complete requests | Full body with all required fields | Partial snippets |
| Show the response | Include response body and status | "Returns the order" |
| Common use cases first | Create → Read → Update → Delete | Alphabetical endpoint list |
| Error examples too | Show what a 422 looks like | Only show success |
| Multiple languages | curl + JS + Python (at minimum) | Only curl |

### Example template (per-endpoint):

```markdown
**curl**

    curl -X POST https://api.example.com/v1/orders \
      -H "Authorization: Bearer sk_test_abc123" \
      -H "Content-Type: application/json" \
      -d '{
        "customer_id": "cust_123",
        "items": [{"product_id": "prod_456", "quantity": 1}]
      }'

**JavaScript**

    const order = await client.orders.create({
      customer_id: 'cust_123',
      items: [{ product_id: 'prod_456', quantity: 1 }]
    });

**Python**

    order = client.orders.create(
        customer_id="cust_123",
        items=[{"product_id": "prod_456", "quantity": 1}]
    )

**Response** `201 Created`

    {
      "id": "ord_789",
      "status": "pending",
      ...
    }
```

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| **No quickstart** | Developers leave before trying the API | Add a 2-minute getting-started example |
| **Auto-generated only** | Technically complete but unusable — no context, no narrative | Write guides and tutorials alongside the reference |
| **Outdated examples** | Examples don't match the current API | Test examples in CI (use spec-driven testing) |
| **"See the code"** | Pointing to source code instead of documenting | Write the docs; source code is not documentation |
| **Jargon-first** | Internal terminology without explanation | Define terms on first use or provide a glossary |
| **No error docs** | Developers can't debug failures | Document every error code with "how to fix" |
| **No versioning info** | Developers don't know which version they're reading | Show version in URL, header, and page title |
| **Giant single page** | 50-page monolith with no navigation | Split into logical sections with a sidebar/TOC |
| **Missing auth section** | Developers can't make their first request | Auth should be the second section after overview |
| **Success-only examples** | Only shows happy path | Show error responses and edge cases |

## Documentation Review Checklist

Before publishing:

- [ ] Can a new developer make a successful API call within 5 minutes?
- [ ] Does every endpoint have a complete request and response example?
- [ ] Are all error codes documented with "how to fix" guidance?
- [ ] Are examples using realistic data, not placeholder strings?
- [ ] Is auth documented before anything else?
- [ ] Are rate limits documented?
- [ ] Is there a changelog with dates?
- [ ] Are deprecated features clearly marked?
- [ ] Have examples been tested against the live API?
- [ ] Are SDKs documented alongside raw HTTP?

## Cross-References

- `api/execution-rest-api-design` — generating the API design that docs describe
- `api/execution-graphql-schema` — GraphQL schema documentation
- `api/knowledge-api-versioning` — deprecation and changelog practices

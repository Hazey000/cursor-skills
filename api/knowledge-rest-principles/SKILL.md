---
name: knowledge-rest-principles
description: >-
  REST architectural constraints, resource modeling, URI design, HTTP method
  semantics, status code selection, content negotiation, and HATEOAS. Includes
  a decision framework for choosing between REST, RPC, and GraphQL. Use when
  designing or reviewing RESTful APIs, or deciding on an API style.
---

# REST Principles & API Style Decision Framework

## Architectural Constraints

REST is defined by six constraints. Violating any of the first five means the API is not RESTful.

| Constraint | What It Means | Violation Smell |
|---|---|---|
| **Client-Server** | UI and data storage concerns are separated | Server rendering client-specific views |
| **Stateless** | Each request contains all information needed to process it | Server-side sessions keyed by cookie |
| **Cacheable** | Responses must declare themselves cacheable or not | Missing `Cache-Control`, `ETag`, `Last-Modified` headers |
| **Uniform Interface** | Resources identified by URIs, manipulated through representations, self-descriptive messages | Verbs in URIs, action-based endpoints |
| **Layered System** | Client cannot tell whether it's connected directly to the server or an intermediary | Hardcoded server addresses, bypassing load balancers |
| **Code-on-Demand** *(optional)* | Server can extend client functionality via executable code | Rarely used in APIs; ignore unless relevant |

## Resource Modeling

Resources are nouns, not actions. Model the domain, not the operations.

**Identify resources by asking:** "What are the things (nouns) in this domain?"

| Domain Concept | Resource | NOT This |
|---|---|---|
| A user account | `/users/{id}` | `/getUser`, `/createUser` |
| An order | `/orders/{id}` | `/placeOrder`, `/cancelOrder` |
| A search result set | `/search?q=term` | `/performSearch` |
| A long-running process | `/jobs/{id}` | `/startProcess` |

### Resource Relationships

Express relationships through URI hierarchy or links:

```
/users/{id}/orders          # orders belonging to a user
/orders/{id}/items          # items within an order
```

Use nesting only for strong ownership (parent must exist). For loose associations, use query filters:

```
/orders?user_id=123         # preferred over deep nesting
```

**Limit nesting to 2 levels max.** Beyond that, promote the sub-resource to a top-level resource.

## URI Design

| Rule | Good | Bad |
|---|---|---|
| Plural nouns for collections | `/users` | `/user`, `/getUsers` |
| Kebab-case | `/order-items` | `/orderItems`, `/order_items` |
| No trailing slashes | `/users` | `/users/` |
| No file extensions | `/users/123` | `/users/123.json` |
| No verbs | `/users` | `/getUsers`, `/deleteUser` |
| IDs for singletons | `/users/123` | `/users?id=123` (for single fetch) |
| Query params for filtering | `/users?role=admin` | `/users/admins` (unless it's a meaningful sub-collection) |

### Action-like operations

When an operation doesn't map cleanly to CRUD, use a sub-resource noun:

```
POST /orders/{id}/cancellation    # not POST /orders/{id}/cancel
POST /users/{id}/password-reset   # not POST /resetPassword
```

Or use a process resource: `POST /jobs` with a `type` field.

## HTTP Method Semantics

| Method | Semantics | Idempotent | Safe | Request Body | Typical Status |
|---|---|---|---|---|---|
| `GET` | Read resource(s) | Yes | Yes | No | 200 |
| `POST` | Create resource or trigger action | No | No | Yes | 201 (created), 202 (async), 200 (action) |
| `PUT` | Full replacement of resource | Yes | No | Yes (complete) | 200 or 204 |
| `PATCH` | Partial update of resource | No* | No | Yes (partial) | 200 |
| `DELETE` | Remove resource | Yes | No | No | 204 or 200 |
| `HEAD` | Same as GET but no body | Yes | Yes | No | 200 |
| `OPTIONS` | Describe communication options | Yes | Yes | No | 200 or 204 |

*PATCH can be made idempotent with `If-Match` / ETags.

### POST vs PUT Decision

- **POST**: Server determines the URI → `POST /users` → `201 Location: /users/123`
- **PUT**: Client determines the URI → `PUT /users/123` → `200` or `201`

### When to Use PATCH vs PUT

- **PUT**: Client sends the complete resource. Omitted fields are reset to defaults.
- **PATCH**: Client sends only changed fields. Use JSON Merge Patch (`application/merge-patch+json`) or JSON Patch (`application/json-patch+json`).

## Status Code Selection

Pick the most specific code. When in doubt, use the decision tree:

```
Is the request valid?
├── No → Is it the client's fault?
│   ├── Auth missing/invalid? → 401
│   ├── Auth valid but insufficient? → 403
│   ├── Resource not found? → 404
│   ├── Method not allowed? → 405
│   ├── Conflict with current state? → 409
│   ├── Validation error? → 422
│   ├── Rate limited? → 429
│   └── Other client error? → 400
└── Yes → Did it succeed?
    ├── Resource created? → 201
    ├── Accepted for async processing? → 202
    ├── No content to return? → 204
    ├── Returning data? → 200
    └── Server error? → 500 (unexpected), 502 (upstream), 503 (overloaded)
```

### Commonly Misused Codes

| Mistake | Correct Usage |
|---|---|
| 200 for creation | 201 + `Location` header |
| 404 when auth fails | 401 (missing) or 403 (forbidden) — use 404 only to hide resource existence |
| 500 for validation errors | 422 or 400 |
| 200 with error body | Use appropriate 4xx/5xx code |

## Content Negotiation

Use `Accept` and `Content-Type` headers, not URL extensions.

```
GET /users/123
Accept: application/json          # client preference
→ Content-Type: application/json  # server response

GET /users/123
Accept: application/xml
→ Content-Type: application/xml   # or 406 if unsupported
```

For API versioning via content negotiation, see `api/knowledge-api-versioning`.

## HATEOAS

**When HATEOAS matters:** Public APIs with many consumers, APIs where discoverability reduces coupling, long-lived APIs where URIs may change.

**When to skip it:** Internal microservice APIs, APIs with a single known consumer, prototyping.

Minimal useful HATEOAS — include `self` and state-transition links:

```json
{
  "id": 123,
  "status": "pending",
  "_links": {
    "self": { "href": "/orders/123" },
    "cancel": { "href": "/orders/123/cancellation", "method": "POST" },
    "payment": { "href": "/orders/123/payment", "method": "POST" }
  }
}
```

## REST vs RPC vs GraphQL Decision Framework

| Factor | REST | RPC (gRPC / JSON-RPC) | GraphQL |
|---|---|---|---|
| **Best for** | Resource-oriented CRUD APIs, public APIs | High-performance service-to-service, action-oriented | Client-driven queries, multiple consumers with different data needs |
| **Data model** | Fixed resources, server-defined shape | Procedures/actions | Flexible, client-specified shape |
| **Over/under-fetching** | Common (mitigated by sparse fieldsets) | Not applicable | Solved by design |
| **Caching** | HTTP caching works naturally | Requires custom caching | Complex (normalized client cache) |
| **Versioning** | Standard patterns (URI, header) | Proto/schema versioning | Schema evolution (additive) |
| **Tooling maturity** | Excellent (OpenAPI, Postman, etc.) | Good (protobuf, gRPC tooling) | Good (Apollo, Relay, GraphiQL) |
| **Learning curve** | Low | Medium | Medium-High |
| **Real-time** | Webhooks, SSE, polling | Bi-directional streaming | Subscriptions |

### Decision Quick-Test

1. **Is it service-to-service with high throughput needs?** → Consider gRPC
2. **Do different clients need very different views of the same data?** → Consider GraphQL
3. **Is it a public API or standard CRUD?** → REST is the default choice
4. **Is the domain action-oriented, not resource-oriented?** → Consider RPC
5. **Uncertain?** → Start with REST. It's the most widely understood.

## Cross-References

- `api/execution-rest-api-design` — step-by-step procedure to design a REST API
- `api/knowledge-api-versioning` — versioning strategies and trade-offs

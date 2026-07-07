---
name: execution-rest-api-design
description: >-
  Step-by-step procedure to design a REST API from requirements to OpenAPI spec.
  Covers resource identification, URI structure, HTTP methods, request/response
  design, error handling, pagination, filtering, and auth planning. Includes
  templates for common patterns. Use when designing a new REST API or extending
  an existing one.
---

# REST API Design Procedure

A repeatable procedure from requirements to a complete, documented API design.

## Prerequisites

- Domain requirements or user stories
- Familiarity with `api/knowledge-rest-principles` (constraints, methods, status codes)
- Auth requirements from stakeholders (see `security/knowledge-auth-patterns` if available)

## Procedure

### Step 1: Identify Resources

Extract nouns from your domain requirements.

**Input:** User stories, domain model, database schema, or conversation context.

**Process:**
1. List every entity/noun mentioned
2. Group related nouns (e.g., "shipping address" → part of `orders` or its own resource?)
3. Determine which are top-level resources vs sub-resources
4. Identify collection vs singleton resources

**Output table:**

| Resource | Type | Parent | Notes |
|---|---|---|---|
| `users` | Collection | — | Core entity |
| `orders` | Collection | — | Belongs to user but accessed independently |
| `order-items` | Collection | `orders` | Always accessed in context of an order |
| `settings` | Singleton | `users` | One per user, no collection |

**Decision rule for sub-resources:** If the child cannot exist without the parent AND is always accessed in the parent's context → sub-resource. Otherwise → top-level with query filter.

### Step 2: Define Relationships

Map how resources relate to each other.

| Relationship | URI Pattern | Example |
|---|---|---|
| Ownership (1:N) | `/parents/{id}/children` | `/users/{id}/orders` |
| Association (M:N) | Link resource or query filter | `/enrollments?user_id=1&course_id=2` |
| Singleton child | `/parents/{id}/child` | `/users/{id}/settings` |

Keep nesting ≤ 2 levels. Promote deeply nested resources to top-level.

### Step 3: Choose URI Structure

Apply naming conventions from `knowledge-rest-principles`:

```
/{version}/{resource-collection}/{resource-id}/{sub-collection}/{sub-id}
```

**Template for each resource:**

```
GET    /v1/orders              # list
POST   /v1/orders              # create
GET    /v1/orders/{id}         # read
PUT    /v1/orders/{id}         # replace
PATCH  /v1/orders/{id}         # partial update
DELETE /v1/orders/{id}         # delete
GET    /v1/orders/{id}/items   # list sub-resource
POST   /v1/orders/{id}/items   # create sub-resource
```

### Step 4: Assign HTTP Methods

For each resource, decide which operations it supports. Not every resource needs all methods.

| Resource | GET (list) | GET (one) | POST | PUT | PATCH | DELETE |
|---|---|---|---|---|---|---|
| `/users` | Yes | Yes | Yes | Yes | Yes | Yes |
| `/users/{id}/settings` | — | Yes | — | Yes | Yes | — |
| `/reports` | Yes | Yes | Yes | — | — | Yes |

For action-like operations, model as sub-resource nouns:

| Action | Endpoint | Method |
|---|---|---|
| Cancel an order | `/orders/{id}/cancellation` | POST |
| Approve a request | `/requests/{id}/approval` | POST |
| Reset password | `/users/{id}/password-reset` | POST |
| Export data | `/exports` | POST (returns 202 + job) |

### Step 5: Design Request/Response Bodies

#### Request Body Template

```json
// POST /v1/orders
{
  "customer_id": "cust_abc123",
  "items": [
    { "product_id": "prod_xyz", "quantity": 2 }
  ],
  "shipping_address": {
    "line_1": "123 Main St",
    "city": "Springfield",
    "postal_code": "62701",
    "country": "US"
  }
}
```

#### Response Body Template

```json
// 201 Created
{
  "id": "ord_789",
  "status": "pending",
  "customer_id": "cust_abc123",
  "items": [
    {
      "id": "item_001",
      "product_id": "prod_xyz",
      "quantity": 2,
      "unit_price": 29.99
    }
  ],
  "total": 59.98,
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-01-15T10:30:00Z"
}
```

**Conventions:**
- Use `snake_case` for field names
- Use ISO 8601 for timestamps with timezone (`Z` or offset)
- Use string IDs with prefixes for readability (`ord_`, `usr_`, `prod_`)
- Include `created_at` and `updated_at` on all mutable resources
- Return the full created/updated resource in responses
- Use consistent envelope vs flat responses (pick one and stick with it)

#### Collection Response Template

```json
{
  "data": [ ... ],
  "pagination": {
    "total": 142,
    "page": 2,
    "per_page": 20,
    "next_cursor": "eyJpZCI6MTIzfQ=="
  }
}
```

### Step 6: Design Error Responses

Use a single consistent error format across all endpoints.

#### Error Response Template

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request body contains invalid fields.",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address.",
        "code": "INVALID_FORMAT"
      },
      {
        "field": "quantity",
        "message": "Must be greater than 0.",
        "code": "OUT_OF_RANGE"
      }
    ],
    "request_id": "req_abc123"
  }
}
```

**Error design rules:**
- Always include a machine-readable `code` (not just the HTTP status)
- Always include a human-readable `message`
- Use `details` array for field-level validation errors
- Include `request_id` for debuggability
- Never expose stack traces, internal paths, or SQL errors

#### Standard Error Codes

Define a finite set of error codes for your API:

| Code | HTTP Status | When |
|---|---|---|
| `NOT_FOUND` | 404 | Resource doesn't exist |
| `VALIDATION_ERROR` | 422 | Request body validation failed |
| `UNAUTHORIZED` | 401 | Missing or invalid credentials |
| `FORBIDDEN` | 403 | Valid credentials, insufficient permissions |
| `CONFLICT` | 409 | State conflict (duplicate, wrong state transition) |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

### Step 7: Add Pagination, Filtering, and Sorting

#### Pagination

Choose one strategy:

| Strategy | Pros | Cons | Use When |
|---|---|---|---|
| **Cursor-based** | Stable under inserts/deletes, performant | Can't jump to page N | Default choice for most APIs |
| **Offset-based** | Simple, supports page jumping | Unstable under concurrent writes | Admin UIs, small datasets |
| **Keyset** | Most performant for large datasets | Requires sortable unique column | High-volume data APIs |

**Cursor pagination template:**

```
GET /v1/orders?limit=20&cursor=eyJpZCI6MTIzfQ==

Response headers:
Link: <https://api.example.com/v1/orders?limit=20&cursor=eyJpZCI6NDU2fQ==>; rel="next"
```

#### Filtering

```
GET /v1/orders?status=pending&created_after=2025-01-01T00:00:00Z&customer_id=cust_abc
```

- Use flat query params for simple filters
- Use field names that match the response body
- Support operators via naming convention if needed: `price_gte=100&price_lte=500`

#### Sorting

```
GET /v1/orders?sort=created_at&order=desc
```

Or multi-field: `sort=status,-created_at` (prefix `-` for descending).

### Step 8: Plan Authentication & Authorization

| Concern | Decision |
|---|---|
| **Auth mechanism** | API key, OAuth 2.0, JWT, or session-based |
| **Where credentials go** | `Authorization: Bearer <token>` (preferred) or API key in header |
| **Scope model** | What permissions exist? Per-resource or role-based? |
| **Rate limiting** | Per API key? Per user? Per endpoint? |
| **CORS** | Which origins? Which methods? Preflight caching? |

See `security/knowledge-auth-patterns` for detailed auth design guidance.

### Step 9: Write the OpenAPI Spec

Translate the design into an OpenAPI 3.x document.

**Minimal spec skeleton:**

```yaml
openapi: 3.1.0
info:
  title: My API
  version: 1.0.0
  description: Brief API description
servers:
  - url: https://api.example.com/v1
paths:
  /orders:
    get:
      summary: List orders
      operationId: listOrders
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [pending, confirmed, shipped, delivered, cancelled]
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
        - name: cursor
          in: query
          schema:
            type: string
      responses:
        '200':
          description: A list of orders
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderList'
    post:
      summary: Create an order
      operationId: createOrder
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created
          headers:
            Location:
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '422':
          $ref: '#/components/responses/ValidationError'
components:
  schemas:
    Order:
      type: object
      required: [id, status, customer_id, items, total, created_at]
      properties:
        id:
          type: string
          example: ord_789
        # ... define all fields
  responses:
    ValidationError:
      description: Validation error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
security:
  - bearerAuth: []
```

**OpenAPI best practices:**
- Use `$ref` for all reusable schemas — never inline complex objects
- Add `example` values to every schema property
- Use `operationId` on every endpoint (drives code generation)
- Define reusable error responses in `components/responses`
- Include `description` on every parameter and schema

### Step 10: Review Checklist

Before finalizing, verify:

- [ ] Every resource has consistent CRUD endpoints (or documented reason for omission)
- [ ] All endpoints return appropriate status codes (not just 200 for everything)
- [ ] Error responses follow a single consistent format
- [ ] Pagination is implemented on all collection endpoints
- [ ] Auth is specified for every endpoint (including which are public)
- [ ] Request/response examples are realistic and complete
- [ ] No verbs in URIs
- [ ] No unnecessary nesting beyond 2 levels
- [ ] `Content-Type` and `Accept` headers are specified
- [ ] Idempotency is considered for POST operations (idempotency keys if needed)
- [ ] Versioning strategy is documented

## Cross-References

- `api/knowledge-rest-principles` — architectural constraints and method semantics
- `api/knowledge-api-versioning` — versioning strategies and deprecation
- `security/knowledge-auth-patterns` — auth mechanism selection
- `api/execution-api-documentation` — documenting the API for consumers

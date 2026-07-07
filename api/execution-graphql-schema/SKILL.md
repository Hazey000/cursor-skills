---
name: execution-graphql-schema
description: >-
  Step-by-step procedure to design a GraphQL schema from a domain model.
  Covers type design, queries, mutations, error handling, pagination (connections
  pattern), subscriptions, and field-level auth. Includes a schema design
  checklist. Use when designing a new GraphQL API or extending an existing schema.
---

# GraphQL Schema Design Procedure

A repeatable procedure from domain model to production-ready GraphQL schema.

## Prerequisites

- Domain model or entity-relationship diagram
- Understanding of which clients will consume the API and their data needs
- Auth requirements (which fields/mutations need authorization)

## Procedure

### Step 1: Map Domain Model to Types

Translate domain entities into GraphQL object types.

**Process:**
1. List domain entities (from database models, domain objects, or requirements)
2. For each entity, list its fields and their types
3. Identify relationships between entities
4. Determine nullability — fields are non-null by default in good schema design; only make nullable when absence is meaningful

**Type design rules:**

| Principle | Do | Don't |
|---|---|---|
| Specific scalars | `DateTime`, `Email`, `URL` (custom scalars) | `String` for everything |
| Enums for finite sets | `enum OrderStatus { PENDING CONFIRMED }` | String with docs saying "must be one of..." |
| Separate input types | `input CreateUserInput { ... }` | Reusing output types as inputs |
| ID types | `ID!` for identifiers | `Int!` or `String!` for IDs |

**Example mapping:**

```graphql
type User {
  id: ID!
  email: String!
  name: String!
  role: UserRole!
  orders(first: Int, after: String): OrderConnection!
  createdAt: DateTime!
}

enum UserRole {
  ADMIN
  MEMBER
  VIEWER
}

type Order {
  id: ID!
  status: OrderStatus!
  items: [OrderItem!]!
  total: Money!
  customer: User!
  createdAt: DateTime!
  updatedAt: DateTime!
}

scalar DateTime
scalar Money
```

### Step 2: Design Queries

Queries are the read surface of your API. Design them around client use cases, not database tables.

**Query naming conventions:**

| Pattern | Example | Use For |
|---|---|---|
| Singular by ID | `user(id: ID!): User` | Fetching a single entity |
| Collection | `users(first: Int, after: String, filter: UserFilter): UserConnection!` | Listing/searching |
| Viewer/Me | `viewer: User!` | Current authenticated user |
| Specialized lookup | `userByEmail(email: String!): User` | Non-ID lookups |

```graphql
type Query {
  # Single entities
  user(id: ID!): User
  order(id: ID!): Order

  # Collections with pagination and filtering
  users(
    first: Int
    after: String
    filter: UserFilter
    orderBy: UserOrderBy
  ): UserConnection!

  orders(
    first: Int
    after: String
    filter: OrderFilter
  ): OrderConnection!

  # Current user context
  viewer: User!
}

input UserFilter {
  role: UserRole
  searchTerm: String
  createdAfter: DateTime
}

enum UserOrderBy {
  CREATED_AT_ASC
  CREATED_AT_DESC
  NAME_ASC
  NAME_DESC
}
```

**Query design rules:**
- Return nullable for single-entity queries (entity may not exist)
- Return non-null connection types for collections (empty list, not null)
- Provide filter input types rather than many top-level query arguments
- Name sort/order enums explicitly (`CREATED_AT_DESC` not just `DESC`)

### Step 3: Design Mutations

Mutations are the write surface. Each mutation should represent a single domain action.

**Mutation design pattern:**

```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
  cancelOrder(input: CancelOrderInput!): CancelOrderPayload!
}

input CreateUserInput {
  email: String!
  name: String!
  role: UserRole = MEMBER
}

type CreateUserPayload {
  user: User
  errors: [UserError!]!
}
```

**Mutation naming conventions:**
- Verb + Noun: `createUser`, `cancelOrder`, `approveRequest`
- Always use a dedicated `Input` type (even for simple mutations)
- Always return a `Payload` type with both the result and errors

**Don't:**
- Use generic names: `updateEntity`, `performAction`
- Return the bare entity type (no error channel)
- Accept loose arguments instead of an input type

### Step 4: Handle Errors

Two main patterns — choose one and be consistent.

#### Pattern A: Errors in Payload (Recommended)

Model domain errors as part of the response type using union types or error fields.

```graphql
type CreateUserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: String
  message: String!
  code: UserErrorCode!
}

enum UserErrorCode {
  EMAIL_TAKEN
  INVALID_EMAIL
  NAME_TOO_SHORT
  UNAUTHORIZED
}
```

**Pros:** Typed errors, client can handle programmatically, works with code generation.

#### Pattern B: Union Result Types

```graphql
union CreateUserResult = CreateUserSuccess | ValidationError | NotFoundError

type CreateUserSuccess {
  user: User!
}

type ValidationError {
  field: String!
  message: String!
}

type NotFoundError {
  message: String!
  resourceType: String!
}
```

**Pros:** Forces clients to handle all cases (like algebraic types). Good for complex error scenarios.

#### What Goes Where

| Error Type | Where It Lives |
|---|---|
| Domain/business errors (email taken, insufficient funds) | In the payload — these are expected outcomes |
| Auth errors (not logged in, insufficient permissions) | GraphQL errors extensions — cross-cutting concern |
| Validation errors (invalid email format) | In the payload `errors` field |
| Infrastructure errors (timeout, unavailable) | GraphQL errors — not recoverable by the client |

### Step 5: Implement Pagination (Connections Pattern)

Use the Relay Connections spec for all collections.

```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int
}

type UserEdge {
  cursor: String!
  node: User!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

**Pagination arguments:**

```graphql
users(
  first: Int     # forward pagination: first N items
  after: String  # forward pagination: after this cursor
  last: Int      # backward pagination: last N items
  before: String # backward pagination: before this cursor
): UserConnection!
```

**Rules:**
- Always provide `PageInfo` with at least `hasNextPage` and `endCursor`
- `totalCount` is optional — expensive on large datasets; omit if you don't need it
- Use opaque cursors (base64-encoded) — clients must not parse or construct them
- Set a maximum page size server-side (e.g., max 100)

### Step 6: Design Subscriptions (If Needed)

Only add subscriptions when clients need real-time updates. Don't add them speculatively.

```graphql
type Subscription {
  orderStatusChanged(orderId: ID!): Order!
  newMessage(conversationId: ID!): Message!
}
```

**Subscription design rules:**
- Subscribe to specific events, not generic "entity changed"
- Include enough data in the payload to avoid follow-up queries
- Consider filtering — let clients subscribe to specific entities, not firehose
- Plan for reconnection: clients should re-subscribe and reconcile state
- Auth applies to subscriptions too — validate on initial connection AND on each event

### Step 7: Plan Field-Level Authorization

Auth in GraphQL typically operates at three levels:

| Level | Implementation | Example |
|---|---|---|
| **Operation** | Middleware/directive on query/mutation | Only admins can call `deleteUser` |
| **Type** | Resolver-level check | Users can only see their own `PaymentMethod` |
| **Field** | Directive or resolver guard | Only the user themselves can see `email` |

**Directive-based approach:**

```graphql
directive @auth(requires: UserRole!) on FIELD_DEFINITION | OBJECT

type User {
  id: ID!
  name: String!
  email: String! @auth(requires: MEMBER)
  role: UserRole! @auth(requires: ADMIN)
}

type Mutation {
  deleteUser(id: ID!): DeleteUserPayload! @auth(requires: ADMIN)
}
```

**Auth design rules:**
- Default to deny — explicitly grant access, don't explicitly restrict
- Log auth failures for security monitoring
- Return `null` for unauthorized fields rather than throwing (unless the field is non-null)
- Document auth requirements in schema descriptions
- Rate-limit mutations independently of queries

### Step 8: Schema Design Checklist

Before finalizing, verify:

**Types & Fields:**
- [ ] All types use specific scalars (DateTime, URL, etc.) not just String
- [ ] Enums are used for all finite value sets
- [ ] Nullability is intentional — non-null is the default, nullable only when absence is meaningful
- [ ] Custom scalars have validation (e.g., Email validates format)
- [ ] `ID` type used for all identifiers

**Queries:**
- [ ] Single-entity lookups return nullable (entity may not exist)
- [ ] Collections use Connections pattern with `PageInfo`
- [ ] Filter/sort options are input types, not loose arguments
- [ ] `viewer`/`me` query exists for authenticated user context

**Mutations:**
- [ ] Each mutation has a dedicated `Input` type and `Payload` type
- [ ] Payload types include an `errors` field for domain errors
- [ ] Mutation names are verb + noun (`createUser` not `user`)
- [ ] No mutation returns a bare type without error channel

**Error Handling:**
- [ ] Domain errors are in payloads, infrastructure errors are in GraphQL errors
- [ ] Error codes are enums, not free-form strings
- [ ] Error messages are user-friendly

**Auth:**
- [ ] Every mutation and sensitive query has auth specified
- [ ] Field-level auth is applied for PII and role-gated data
- [ ] Auth failures are logged
- [ ] Rate limiting is configured per-operation

**Performance:**
- [ ] N+1 queries addressed (DataLoader or equivalent)
- [ ] Query depth limiting is configured
- [ ] Query complexity analysis is configured
- [ ] Maximum page sizes are enforced server-side

## Cross-References

- `architecture/knowledge-ddd` — domain-driven design for modeling types from bounded contexts
- `api/knowledge-rest-principles` — REST vs GraphQL decision framework
- `api/execution-api-documentation` — documenting the schema for consumers

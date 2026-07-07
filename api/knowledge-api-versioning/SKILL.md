---
name: knowledge-api-versioning
description: >-
  API versioning strategies (URI path, query param, header, content negotiation)
  with trade-offs matrix. Breaking vs non-breaking changes, deprecation policies,
  Sunset headers, and migration strategies. Includes a decision framework for
  choosing a versioning approach. Use when planning API versioning or managing
  breaking changes.
---

# API Versioning

## Versioning Strategies

### Trade-Offs Matrix

| Strategy | Example | Visibility | Cache-Friendly | Proxy-Friendly | Client Complexity | Best For |
|---|---|---|---|---|---|---|
| **URI Path** | `/v1/users` | Explicit | Yes | Yes | Low | Public APIs, simple versioning |
| **Query Param** | `/users?version=1` | Explicit | Conditional | Yes | Low | Optional versioning, gradual rollout |
| **Custom Header** | `X-API-Version: 1` | Hidden | No (varies) | Fragile | Medium | Internal APIs, fine-grained control |
| **Content Negotiation** | `Accept: application/vnd.api.v1+json` | Hidden | Yes (Vary) | Fragile | High | Mature APIs, per-resource versioning |

### URI Path Versioning

```
GET /v1/users/123
GET /v2/users/123
```

- Version the entire API surface at once
- Simple routing at gateway/load balancer level
- Downside: even unchanged resources get a new URI, breaks bookmarks and `_links`
- Convention: `v{major}` only — don't encode minor/patch in the URI

### Query Parameter Versioning

```
GET /users/123?version=2
```

- Resource URI stays stable
- Version can default to latest (or pinned)
- Downside: easy to forget, not all caching layers handle query params well
- Best used when versioning is opt-in and most clients use the latest

### Custom Header Versioning

```
GET /users/123
X-API-Version: 2
```

- Clean URIs, no path pollution
- Downside: invisible in browser, harder to test/debug, some proxies strip custom headers
- Always use a consistent header name across all services

### Content Negotiation (Media Type Versioning)

```
GET /users/123
Accept: application/vnd.myapi.v2+json
```

- Most RESTful approach — version is part of the representation, not the resource
- Allows per-resource versioning (user v2 but orders still v1)
- Downside: complex for clients, poor tooling support, hard to discover
- Use `Vary: Accept` for correct caching

## Decision Framework

```
Is this a public API with many consumers?
├── Yes → URI Path versioning (simplest for external developers)
└── No → Is it an internal microservice API?
    ├── Yes → Do you need per-resource versioning?
    │   ├── Yes → Content Negotiation
    │   └── No → Custom Header (or just don't version — use additive changes)
    └── No → Is versioning opt-in?
        ├── Yes → Query Parameter (default to latest)
        └── No → URI Path
```

**Rule of thumb:** If you're unsure, use URI path versioning. It's the most widely understood and tooling-friendly approach.

## Breaking vs Non-Breaking Changes

### Non-Breaking (Safe) Changes

These do NOT require a version bump:

- Adding a new optional field to a response
- Adding a new endpoint
- Adding a new optional query parameter
- Adding a new enum value (if clients handle unknown values)
- Adding a new HTTP method to an existing resource
- Relaxing a validation constraint (accepting wider input)
- Adding new optional request body fields

### Breaking Changes

These REQUIRE a version bump or migration strategy:

- Removing or renaming a response field
- Removing an endpoint
- Changing a field's type (string → int)
- Changing a field from optional to required
- Changing the meaning/semantics of a field
- Tightening a validation constraint
- Changing error response format
- Changing authentication mechanism
- Changing pagination format

### Grey Area

Requires judgement — check client impact:

| Change | Breaking? | Depends On |
|---|---|---|
| Adding a required request field | Usually yes | Whether existing clients send it |
| Changing default sort order | Maybe | Whether clients rely on order |
| Adding rate limits | Maybe | Whether clients handle 429 |
| Changing response field from non-null to nullable | Usually yes | Whether clients handle null |

## Deprecation Policy

### Timeline

A deprecation policy should define:

1. **Announcement**: Notify consumers before any deprecation (minimum 3 months for public APIs)
2. **Deprecation period**: Both versions run simultaneously (minimum 6 months for public APIs)
3. **Sunset**: Old version is removed

### Sunset Header (RFC 8594)

Signal deprecation and upcoming removal in HTTP responses:

```http
HTTP/1.1 200 OK
Sunset: Sat, 01 Mar 2025 00:00:00 GMT
Deprecation: true
Link: <https://api.example.com/docs/migration-v3>; rel="sunset"
```

| Header | Purpose |
|---|---|
| `Sunset` | Date after which the endpoint may be removed |
| `Deprecation: true` | Marks the response as deprecated |
| `Link` with `rel="sunset"` | Points to migration documentation |

### Deprecation Checklist

- [ ] Document all breaking changes between versions
- [ ] Publish a migration guide with before/after examples
- [ ] Add `Sunset` and `Deprecation` headers to old version responses
- [ ] Set up monitoring/alerting on old version traffic
- [ ] Notify consumers via changelog, email, developer portal
- [ ] Log which consumers still use the deprecated version
- [ ] Provide a migration tool or compatibility shim if feasible
- [ ] Set a hard sunset date and communicate it clearly

## Migration Strategies

### Parallel Running

Run both versions simultaneously behind the same gateway. Route by version identifier.

```
api.example.com/v1/users  →  v1 handler
api.example.com/v2/users  →  v2 handler
```

- Simplest to implement
- Higher infrastructure cost (two codepaths or services)
- Use when breaking changes are significant

### Adapter/Translation Layer

Keep one canonical internal representation. Translate at the API boundary.

```
Request (v1 format) → Adapter → Canonical → Handler → Adapter → Response (v1 format)
Request (v2 format) → ────────→ Canonical → Handler → ────────→ Response (v2 format)
```

- Single business logic, multiple API shapes
- Good for format changes (field renames, restructuring)
- Adapter complexity grows with version count — limit to 2–3 active versions

### Expand-Contract (Additive Migration)

Avoid versioning altogether for many changes:

1. **Expand**: Add new fields alongside old ones, populate both
2. **Migrate**: Consumers move to new fields at their own pace
3. **Contract**: Remove old fields after all consumers have migrated

```json
// Phase 1: Expand — both fields present
{ "name": "Alice", "full_name": "Alice Smith" }

// Phase 3: Contract — old field removed
{ "full_name": "Alice Smith" }
```

Best when: you control consumers or can track field usage.

### Feature Flags / Gradual Rollout

Use feature flags to expose new behaviour to specific consumers before a full version bump.

```
GET /users/123
X-Feature-Flags: new-pagination
```

- Good for testing breaking changes with friendly consumers first
- Not a substitute for proper versioning on public APIs

## Version Lifecycle Management

| State | Meaning | Actions |
|---|---|---|
| **Current** | Latest stable version | Full support, actively developed |
| **Supported** | Previous version still in use | Bug fixes and security patches only |
| **Deprecated** | Sunset date announced | `Sunset` header set, migration docs published |
| **Retired** | No longer available | Returns 410 Gone with migration pointer |

**Best practice:** Support at most 2–3 concurrent versions. More creates an unsustainable maintenance burden.

## Cross-References

- `api/knowledge-rest-principles` — foundational REST constraints
- `api/execution-rest-api-design` — end-to-end API design procedure including versioning step

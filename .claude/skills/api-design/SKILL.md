---
name: api-design
description: REST API design patterns including resource naming, status codes, pagination, filtering, error responses, versioning, and rate limiting for production APIs.
origin: ECC
---

# API Design Patterns

Conventions and best practices for designing consistent, developer-friendly REST APIs.

## When to Activate

- Designing new API endpoints
- Reviewing existing API contracts
- Adding pagination, filtering, or sorting
- Implementing error handling for APIs
- Planning API versioning strategy
- Building public or partner-facing APIs

## Resource Design

### URL Structure

```
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# Sub-resources for relationships
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# Actions that don't map to CRUD (use verbs sparingly)
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
```

### Naming Rules

```
# GOOD
/api/v1/team-members          # kebab-case for multi-word resources
/api/v1/orders?status=active  # query params for filtering

# BAD
/api/v1/getUsers              # verb in URL
/api/v1/user                  # singular (use plural)
/api/v1/team_members          # snake_case in URLs
```

## HTTP Status Codes

```
200 OK                    — GET, PUT, PATCH (with body)
201 Created               — POST (include Location header)
204 No Content            — DELETE, PUT (no body)
400 Bad Request           — Validation failure, malformed JSON
401 Unauthorized          — Missing or invalid authentication
403 Forbidden             — Authenticated but not authorized
404 Not Found             — Resource doesn't exist
409 Conflict              — Duplicate entry, state conflict
422 Unprocessable Entity  — Valid JSON, semantically invalid
429 Too Many Requests     — Rate limit exceeded
500 Internal Server Error — Never expose internal details
```

## Response Format

### Success

```json
{ "data": { "id": "abc-123", "email": "alice@example.com" } }
```

### Collection with Pagination

```json
{
  "data": [...],
  "meta": { "total": 142, "page": 1, "per_page": 20, "total_pages": 8 },
  "links": { "self": "...", "next": "...", "last": "..." }
}
```

### Error

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [{ "field": "email", "message": "Must be a valid email", "code": "invalid_format" }]
  }
}
```

## Pagination

**Cursor-based (recommended for large datasets):**
```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20
```

**Offset-based (simple, for small datasets):**
```
GET /api/v1/users?page=2&per_page=20
```

## Filtering & Sorting

```
GET /api/v1/orders?status=active&customer_id=abc
GET /api/v1/products?price[gte]=10&price[lte]=100
GET /api/v1/products?sort=-created_at,price
```

## Rate Limiting Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

## Versioning Strategy

- Use URL path versioning: `/api/v1/`, `/api/v2/`
- Maintain at most 2 active versions
- Adding fields/endpoints = non-breaking (no new version needed)
- Removing/renaming fields = breaking (new version required)
- Give 6 months deprecation notice for public APIs

## API Design Checklist

- [ ] Resource URL follows naming conventions (plural, kebab-case, no verbs)
- [ ] Correct HTTP method and status codes used
- [ ] Input validated with schema (Zod, Pydantic, etc.)
- [ ] Error responses follow standard format
- [ ] Pagination implemented for list endpoints
- [ ] Authentication and authorization enforced
- [ ] Rate limiting configured
- [ ] Response does not leak internal details

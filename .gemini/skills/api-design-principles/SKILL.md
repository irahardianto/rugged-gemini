---
name: api-design-principles
description: >-
  REST/HTTP API design: resource naming, status codes, error formats,
  versioning, pagination.
user-invocable: false
---

## API Design Principles

### RESTful Standards

**URLs:** plural nouns (`/api/v1/users`), hierarchical (`/users/:id/orders`), no verbs.

**Methods:** GET=read (safe, idempotent, cacheable), POST=create, PUT=replace (idempotent), PATCH=partial update (idempotent), DELETE=remove (idempotent).

**Versioning:** URL path `/api/v1/users`.

**Pagination:** default 20, max 100. Cursor (`?cursor=abc123`) or offset (`?page=2&limit=20`).

**Filter/Sort:** `?status=active&role=admin`, `?sort=created_at:desc,name:asc`, `?q=search+term`.

### Status Codes

**Success:** 200 OK (GET/PUT/PATCH), 201 Created (POST), 204 No Content (DELETE).

**4xx (client can fix):**
- 400 Bad Request — validation (invalid email, missing field). Detailed field errors.
- 401 Unauthorized — auth failed (invalid/expired/missing token)
- 403 Forbidden — permission denied (identified but lacks access)
- 404 Not Found — doesn't exist (or permission to know it exists)
- 409/422 — business rule violation (duplicate, insufficient balance)
- 429 — rate limited. Wait + retry.

**5xx (system):** 500/502/503 — generic message + correlationId. User: retry later.

### Success Response
```
{
  "data": { /* resource or array */ },
  "meta": {
    "total": 100,
    "page": 1,
    "perPage": 20
  },
  "links": {
    "self": "/api/v1/users?page=1",
    "next": "/api/v1/users?page=2",
    "prev": null
  }
}
```

### Error Response
```
{
  "status": "error",
  "code": 400,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "reason": "Must be a valid address"
    },
  "correlationId": "req-1234567890",
  "doc_url": "https://..."
  }
}
```

### Related
- Error Handling GEMINI.md § Error Handling Principles
- Security Mandate GEMINI.md § Security Mandate
- Security Principles GEMINI.md § Security Principles
- Logging Mandate GEMINI.md § Logging and Observability Mandate
- Data Serialization @.gemini/skills/data-serialization-principles/SKILL.md
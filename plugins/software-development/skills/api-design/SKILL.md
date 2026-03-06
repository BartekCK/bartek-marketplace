---
name: api-design
description: "REST API design conventions and best practices for TypeScript/Node.js. Loaded by backend-software-developer-agent when creating endpoints, designing request/response schemas, or structuring API routes."
user-invocable: false
---

# REST API Design — TypeScript Cheat Sheet

## Resource Naming

- Use **plural nouns**: `/users`, `/orders`, `/order-items`
- Nest for relationships: `/users/:userId/orders`
- Use kebab-case for multi-word resources: `/payment-methods`
- Avoid verbs in URLs — let HTTP methods convey the action
- Keep nesting shallow (max 2 levels); use query params or filters beyond that

## HTTP Methods & Status Codes

| Method | Purpose | Success | Body |
|--------|---------|---------|------|
| GET | Read resource(s) | 200 | Resource / array |
| POST | Create resource | 201 | Created resource + `Location` header |
| PUT | Full replace | 200 | Updated resource |
| PATCH | Partial update | 200 | Updated resource |
| DELETE | Remove | 204 | No body |

### Error Status Codes

| Code | Meaning |
|------|---------|
| 400 | Validation / bad input |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict (duplicate, stale update) |
| 422 | Semantically invalid (parseable but wrong) |
| 500 | Unexpected server error |

## Typed Request/Response Interfaces

```typescript
// Shared types
interface ApiError {
  status: number;
  code: string;           // machine-readable: "VALIDATION_ERROR"
  message: string;        // human-readable
  details?: Record<string, string[]>;  // field-level errors
}

interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    pageSize: number;
    totalPages: number;
  };
}

// Resource DTOs
interface CreateUserDto {
  email: string;
  name: string;
  role: "admin" | "member";
}

interface UserResponse {
  id: string;
  email: string;
  name: string;
  role: string;
  createdAt: string;   // ISO 8601
}
```

## Error Response Format

Always return consistent error shape:

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid input",
  "details": {
    "email": ["Must be a valid email address"],
    "name": ["Required"]
  }
}
```

## Pagination

Use query parameters: `GET /users?page=2&pageSize=20`

```typescript
function paginate<T>(items: T[], total: number, page: number, pageSize: number): PaginatedResponse<T> {
  return {
    data: items,
    meta: { total, page, pageSize, totalPages: Math.ceil(total / pageSize) },
  };
}
```

For large datasets, prefer **cursor-based** pagination:

```
GET /events?cursor=abc123&limit=50
{ data: [...], meta: { nextCursor: "def456", hasMore: true } }
```

## Versioning

Prefer **URL prefix** for simplicity: `/api/v1/users`. Alternatives (Accept header) add complexity with little benefit for most projects.

## Typed Route Handler (Express)

```typescript
import { Request, Response, NextFunction } from "express";

interface TypedRequest<B = unknown, Q = unknown> extends Request {
  body: B;
  query: Q;
}

const createUser = async (
  req: TypedRequest<CreateUserDto>,
  res: Response<UserResponse | ApiError>,
  next: NextFunction
) => {
  try {
    const user = await userService.create(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
};

router.post("/users", validateBody(createUserSchema), createUser);
```

## Typed Route Handler (Fastify)

```typescript
import { FastifyRequest, FastifyReply } from "fastify";

app.post<{ Body: CreateUserDto; Reply: UserResponse }>(
  "/users",
  { schema: { body: createUserSchema, response: { 201: userResponseSchema } } },
  async (request, reply) => {
    const user = await userService.create(request.body);
    reply.status(201).send(user);
  }
);
```

## Quick Rules

- Validate input at the boundary (zod, ajv, or framework schema)
- Return `Content-Type: application/json` consistently
- Use ISO 8601 for dates, UUIDs for IDs
- Idempotency: PUT and DELETE should be safe to retry
- Rate limit public endpoints; return `429` with `Retry-After` header
- Document with OpenAPI/Swagger — generate types from schema where possible

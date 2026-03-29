---
name: typescript-strict-typing
description: Strict TypeScript typing patterns and anti-patterns for full-stack development. Loaded by both backend-software-developer-agent and frontend-software-developer-agent to enforce type safety and eliminate any/undefined usage.
user-invocable: false
disable-model-invocation: true
---

# Strict TypeScript Typing Cheat Sheet

## Hard Rules

1. **NEVER use `any`** — use `unknown` and narrow with type guards.
2. **NEVER leave implicit `undefined`** — use optional `?` or explicit `| null` union.
3. **Always type function return values** — no inferred returns on exported/public functions.
4. **Use `as const`** for literal values and config objects.
5. **Prefer discriminated unions** over type assertions.

## Anti-Patterns and Corrections

### `any` to proper type

```typescript
// BAD
function parse(input: any) { return input.name; }

// GOOD
function parse(input: unknown): string {
  if (typeof input === "object" && input !== null && "name" in input) {
    return (input as { name: string }).name;
  }
  throw new Error("Invalid input");
}
```

### `as Type` to type guard

```typescript
// BAD
const user = data as User;

// GOOD
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data && "email" in data;
}
if (isUser(data)) { /* data is User here */ }
```

### Non-null assertion `!` to proper check

```typescript
// BAD
const el = document.getElementById("root")!;

// GOOD
const el = document.getElementById("root");
if (!el) throw new Error("Root element not found");
```

## Utility Types

```typescript
// Pick subset of properties
type UserSummary = Pick<User, "id" | "name">;

// Omit properties
type CreateUserDto = Omit<User, "id" | "createdAt">;

// Make all optional (for PATCH updates)
type UpdateUserDto = Partial<Pick<User, "name" | "email">>;

// Make all required
type FullConfig = Required<Config>;

// Record for typed dictionaries
type RolePermissions = Record<Role, Permission[]>;

// Extract union member
type AdminEvent = Extract<AppEvent, { type: "admin.created" | "admin.deleted" }>;
```

## Discriminated Unions

Use a shared literal `type` field to let TypeScript narrow automatically.

```typescript
type ApiResult =
  | { status: "success"; data: User }
  | { status: "error"; error: string; code: number }
  | { status: "loading" };

function handle(result: ApiResult) {
  switch (result.status) {
    case "success": return result.data; // TypeScript knows `data` exists
    case "error": throw new Error(result.error); // TypeScript knows `error` exists
    case "loading": return null;
  }
}
```

## Branded Types

Prevent mixing structurally identical types (e.g., two `string` IDs).

```typescript
type Brand<T, B extends string> = T & { readonly __brand: B };
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

function createUserId(id: string): UserId { return id as UserId; }
function getUser(id: UserId): User { /* ... */ }

const userId = createUserId("abc");
const orderId = "abc" as OrderId;
// getUser(orderId); // ERROR: OrderId is not assignable to UserId
```

## Zod Schema Inference

Define the schema once, infer the TypeScript type from it.

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(["admin", "user", "viewer"]),
  metadata: z.record(z.string(), z.unknown()).optional(),
});

type User = z.infer<typeof UserSchema>;

// Validation
function parseUser(input: unknown): User {
  return UserSchema.parse(input); // throws ZodError on failure
}
```

## `as const` for Literals

```typescript
// Without: type is string[]
const ROLES = ["admin", "user", "viewer"];

// With: type is readonly ["admin", "user", "viewer"]
const ROLES = ["admin", "user", "viewer"] as const;
type Role = (typeof ROLES)[number]; // "admin" | "user" | "viewer"
```

## Generic Constraints

```typescript
// Ensure T has an id property
function findById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find((item) => item.id === id);
}

// Ensure key exists on object
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

## Typing Async Functions

```typescript
// Always type the return
async function fetchUsers(): Promise<User[]> {
  const res = await fetch("/api/users");
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return UserSchema.array().parse(await res.json());
}
```

## Key Rules Summary

- `unknown` over `any` — always.
- Type guards (`is`) over assertions (`as`) — always.
- Discriminated unions over type predicates for state modeling.
- Zod for runtime validation, infer static types from schemas.
- Branded types for domain IDs to prevent cross-assignment.
- Explicit return types on all exported functions.

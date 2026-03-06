---
name: error-handling
description: Error handling patterns for TypeScript applications — custom errors, Result types, React error boundaries, and API error responses. Loaded by both backend-software-developer-agent and frontend-software-developer-agent when implementing error handling strategies.
user-invocable: false
---

# Error Handling Cheat Sheet

## Backend: Custom Error Hierarchy

Define a base error class with an HTTP status code, then extend for specific cases.

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly statusCode: number,
    public readonly code: string,
    public readonly details?: Record<string, unknown>,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`, 404, "NOT_FOUND", { resource, id });
  }
}

class ValidationError extends AppError {
  constructor(errors: Array<{ field: string; message: string }>) {
    super("Validation failed", 400, "VALIDATION_ERROR", { errors });
  }
}

class AuthError extends AppError {
  constructor(message = "Unauthorized") {
    super(message, 401, "UNAUTHORIZED");
  }
}

class ForbiddenError extends AppError {
  constructor(message = "Forbidden") {
    super(message, 403, "FORBIDDEN");
  }
}
```

## Backend: Typed Error Response

```typescript
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
  };
}
```

## Backend: Express Error Middleware

```typescript
function errorHandler(err: Error, req: Request, res: Response, _next: NextFunction): void {
  // Log with context — never swallow errors
  console.error({ err, method: req.method, path: req.path, body: req.body });

  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: { code: err.code, message: err.message, details: err.details },
    } satisfies ErrorResponse);
    return;
  }

  // Unknown errors — never expose internals
  res.status(500).json({
    error: { code: "INTERNAL_ERROR", message: "An unexpected error occurred" },
  } satisfies ErrorResponse);
}

// Register after all routes
app.use(errorHandler);
```

## Backend: Usage in Route Handlers

```typescript
async function getUser(req: Request, res: Response, next: NextFunction) {
  try {
    const user = await db.users.findUnique({ where: { id: req.params.id } });
    if (!user) throw new NotFoundError("User", req.params.id);
    res.json(user);
  } catch (err) {
    next(err); // Forward to error middleware
  }
}
```

## Shared: Result Type Pattern

Avoid try-catch for expected failures. Model success and failure explicitly.

```typescript
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

function ok<T>(value: T): Result<T, never> {
  return { ok: true, value };
}

function err<E>(error: E): Result<never, E> {
  return { ok: false, error };
}

// Usage
async function findUser(id: string): Promise<Result<User, NotFoundError>> {
  const user = await db.users.findUnique({ where: { id } });
  if (!user) return err(new NotFoundError("User", id));
  return ok(user);
}

const result = await findUser("123");
if (!result.ok) {
  // handle result.error — fully typed
  return;
}
// result.value is User here
```

## Shared: Typed try-catch

The `catch` parameter is always `unknown`. Narrow before using.

```typescript
try {
  await riskyOperation();
} catch (error: unknown) {
  if (error instanceof AppError) {
    logger.warn({ code: error.code, message: error.message });
  } else if (error instanceof Error) {
    logger.error({ message: error.message, stack: error.stack });
  } else {
    logger.error({ message: "Unknown error", error });
  }
}
```

## Frontend: React Error Boundary

```tsx
import { Component, type ErrorInfo, type ReactNode } from "react";

interface Props { children: ReactNode; fallback: ReactNode }
interface State { hasError: boolean }

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State { return { hasError: true }; }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error("ErrorBoundary caught:", error, info.componentStack);
    // Send to error tracking service (Sentry, etc.)
  }

  render() { return this.state.hasError ? this.props.fallback : this.props.children; }
}

// Usage — one boundary per route or feature section
<ErrorBoundary fallback={<p>Something went wrong.</p>}>
  <UserProfile />
</ErrorBoundary>
```

## Frontend: Async Error Handling in Event Handlers

Error boundaries do not catch errors in event handlers. Handle them explicitly.

```tsx
function DeleteButton({ id }: { id: string }) {
  const [error, setError] = useState<string | null>(null);

  async function handleDelete() {
    try {
      await api.deleteUser(id);
      toast.success("User deleted");
    } catch (err: unknown) {
      const message = err instanceof Error ? err.message : "Delete failed";
      setError(message);
      toast.error(message);
    }
  }

  return <button onClick={handleDelete}>Delete</button>;
}
```

## Key Rules

- **Never swallow errors** — always log with request/operation context.
- **Never expose internal details** to clients — map to safe error responses.
- **Use Result type** for expected failures (not found, validation).
- **Use exceptions** for unexpected failures (network, database).
- **One Error Boundary per route** minimum; add more around risky features.
- **Typed catch blocks** — always narrow `unknown` before accessing properties.

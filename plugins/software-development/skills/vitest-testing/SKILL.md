---
name: vitest-testing
description: Vitest testing patterns and conventions for TypeScript projects. This skill should be used when the project uses Vitest as its test framework — detected via vitest in package.json dependencies, vitest.config.ts, or vite.config.ts with test configuration. Loaded by the tester-agent when creating or modifying tests in a Vitest project.
user-invocable: false
---

# Vitest Testing Conventions

## Setup and Configuration

Detect Vitest configuration in this order:
1. `vitest.config.ts` — standalone config
2. `vite.config.ts` — with `test` field
3. `package.json` — `vitest` in dependencies

Read the config to understand: test file patterns, setup files, coverage settings, environment (`jsdom` for frontend, `node` for backend).

## Imports

Vitest requires explicit imports:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
```

## Mocking

### Function Mocks

```typescript
const mockFn = vi.fn();
const mockFnTyped = vi.fn<[string, number], boolean>();

// Return values
mockFn.mockReturnValue("value");
mockFn.mockResolvedValue({ id: "1" });
mockFn.mockRejectedValue(new Error("fail"));

// Implementation
mockFn.mockImplementation((a, b) => a + b);
```

### Module Mocking

```typescript
// Auto-mock entire module
vi.mock("./userRepository");

// Manual mock with factory
vi.mock("./userRepository", () => ({
  UserRepository: vi.fn().mockImplementation(() => ({
    findById: vi.fn(),
    save: vi.fn(),
  })),
}));

// Partial mock — keep real implementations, override specific exports
vi.mock("./utils", async (importOriginal) => {
  const actual = await importOriginal<typeof import("./utils")>();
  return {
    ...actual,
    generateId: vi.fn().mockReturnValue("mock-id"),
  };
});
```

### Spy on Object Methods

```typescript
const spy = vi.spyOn(service, "process");
spy.mockResolvedValue(result);

// Verify calls
expect(spy).toHaveBeenCalledWith("arg1");
expect(spy).toHaveBeenCalledTimes(1);
```

### Reset Between Tests

```typescript
beforeEach(() => {
  vi.clearAllMocks();   // Clear call history and results
  // or
  vi.resetAllMocks();   // Clear + remove implementations
  // or
  vi.restoreAllMocks(); // Reset + restore originals (for spies)
});
```

## Timers

```typescript
beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

it("should debounce", async () => {
  // Given
  const callback = vi.fn();
  const debounced = debounce(callback, 300);

  // When
  debounced();
  vi.advanceTimersByTime(300);

  // Then
  expect(callback).toHaveBeenCalledOnce();
});
```

## Async Testing

```typescript
// Resolved values
it("should fetch user", async () => {
  // Given
  mockRepo.findById.mockResolvedValue({ id: "1", name: "John" });

  // When
  const result = await service.getUser("1");

  // Then
  expect(result).toEqual({ id: "1", name: "John" });
});

// Rejected / error paths
it("should throw on not found", async () => {
  // Given
  mockRepo.findById.mockResolvedValue(null);

  // When / Then
  await expect(service.getUser("999")).rejects.toThrow(NotFoundError);
});
```

## Snapshot Testing

Use sparingly — only for complex serializable outputs where manual assertions are impractical:

```typescript
it("should render config", () => {
  // Given
  const input = { theme: "dark", locale: "en" };

  // When
  const config = generateConfig(input);

  // Then
  expect(config).toMatchInlineSnapshot(`
    {
      "theme": "dark",
      "locale": "en",
      "version": "1.0.0",
    }
  `);
});
```

Prefer `toMatchInlineSnapshot` over `toMatchSnapshot` for visibility.

## React Component Testing (with @testing-library/react)

```typescript
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("should submit the form", async () => {
  // Given
  const onSubmit = vi.fn();
  const user = userEvent.setup();
  render(<LoginForm onSubmit={onSubmit} />);

  // When
  await user.type(screen.getByLabelText("Email"), "john@example.com");
  await user.type(screen.getByLabelText("Password"), "secret123");
  await user.click(screen.getByRole("button", { name: "Sign in" }));

  // Then
  expect(onSubmit).toHaveBeenCalledWith({
    email: "john@example.com",
    password: "secret123",
  });
});
```

Use `userEvent` over `fireEvent` — it simulates real user behavior (typing, clicking) more accurately.

## Running Tests

```bash
# Run specific test file
npx vitest run path/to/file.test.ts

# Run with pattern matching
npx vitest run --reporter=verbose -t "should create user"

# Run in watch mode (development)
npx vitest path/to/file.test.ts

# Run with coverage
npx vitest run --coverage
```

## Key Vitest-Specific Features

- **In-source testing**: `if (import.meta.vitest)` blocks (avoid in production code)
- **Type testing**: `expectTypeOf<string>().toMatchTypeOf<string>()` — for compile-time type assertions
- **Concurrent tests**: `it.concurrent("test", async () => {})` — run tests in parallel within a describe
- **Benchmarking**: `bench("name", () => {})` — built-in performance benchmarks (separate from test suites)

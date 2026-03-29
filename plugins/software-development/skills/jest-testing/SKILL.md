---
name: jest-testing
description: "Jest testing patterns and conventions for TypeScript projects. This skill should be used when the project uses Jest as its test framework — detected via jest in package.json dependencies, jest.config.ts, jest.config.js, or a jest field in package.json. Loaded by the tester-agent when creating or modifying tests in a Jest project."
user-invocable: false
disable-model-invocation: true
---

# Jest Testing Conventions

## Setup and Configuration

Detect Jest configuration in this order:
1. `jest.config.ts` or `jest.config.js` — standalone config
2. `package.json` — `jest` field
3. `package.json` — `jest` in dependencies

Read the config to understand: test file patterns, setup files, module name mapping, transform settings, test environment (`jsdom` for frontend, `node` for backend).

## Imports

Jest globals are available without imports by default. If using `@jest/globals`:

```typescript
import { describe, it, expect, jest, beforeEach, afterEach } from "@jest/globals";
```

Check the project's `tsconfig.json` for `"types": ["jest"]` and existing test files for the import convention.

## Mocking

### Function Mocks

```typescript
const mockFn = jest.fn();
const mockFnTyped = jest.fn<(a: string, b: number) => boolean>();

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
jest.mock("./userRepository");

// Manual mock with factory
jest.mock("./userRepository", () => ({
  UserRepository: jest.fn().mockImplementation(() => ({
    findById: jest.fn(),
    save: jest.fn(),
  })),
}));
```

Note: Jest hoists `jest.mock()` calls to the top of the file automatically. Factory functions cannot reference variables defined in the test file unless prefixed with `mock`:

```typescript
// This works — variable name starts with "mock"
const mockFindById = jest.fn();
jest.mock("./userRepository", () => ({
  UserRepository: jest.fn(() => ({ findById: mockFindById })),
}));
```

### Spy on Object Methods

```typescript
const spy = jest.spyOn(service, "process");
spy.mockResolvedValue(result);

// Verify calls
expect(spy).toHaveBeenCalledWith("arg1");
expect(spy).toHaveBeenCalledTimes(1);
```

### Reset Between Tests

```typescript
beforeEach(() => {
  jest.clearAllMocks();   // Clear call history and results
  // or
  jest.resetAllMocks();   // Clear + remove implementations
  // or
  jest.restoreAllMocks(); // Reset + restore originals (for spies)
});
```

## Timers

```typescript
beforeEach(() => {
  jest.useFakeTimers();
});

afterEach(() => {
  jest.useRealTimers();
});

it("should debounce", () => {
  // Given
  const callback = jest.fn();
  const debounced = debounce(callback, 300);

  // When
  debounced();
  jest.advanceTimersByTime(300);

  // Then
  expect(callback).toHaveBeenCalledTimes(1);
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

Use sparingly — only for complex serializable outputs:

```typescript
it("should render config", () => {
  // Given
  const input = { theme: "dark", locale: "en" };

  // When
  const config = generateConfig(input);

  // Then
  expect(config).toMatchInlineSnapshot(`
    Object {
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
  const onSubmit = jest.fn();
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

Use `userEvent` over `fireEvent` — it simulates real user behavior more accurately.

## Running Tests

```bash
# Run specific test file
npx jest path/to/file.test.ts

# Run with pattern matching
npx jest --verbose -t "should create user"

# Run in watch mode (development)
npx jest --watch path/to/file.test.ts

# Run with coverage
npx jest --coverage
```

## TypeScript with Jest

Jest requires a transform for TypeScript. Common setups:

- **ts-jest**: `transform: { "^.+\\.tsx?$": "ts-jest" }` — uses the TypeScript compiler
- **@swc/jest**: `transform: { "^.+\\.tsx?$": ["@swc/jest"] }` — faster, uses SWC
- **babel-jest**: with `@babel/preset-typescript` — strips types only, no type checking

Check the project's existing transform configuration before writing tests. Match the import style (`ESM` vs `CJS`) accordingly.

## Key Jest-Specific Behaviors

- **Mock hoisting**: `jest.mock()` is hoisted above imports — factory variables must be prefixed with `mock`
- **Automatic resetting**: Configure `clearMocks: true` in jest.config to auto-clear between tests
- **Module isolation**: `jest.isolateModules(() => {})` for tests that need fresh module state
- **Manual mocks**: `__mocks__/` directory next to `node_modules/` or next to the mocked file for automatic resolution
- **Each callback scope**: In `beforeAll`/`beforeEach` callbacks, `this` is shared within the describe block (rarely used with TypeScript)

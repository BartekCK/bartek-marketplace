---
name: solid-principles
description: "SOLID principles reference for TypeScript/Node.js backend development. Loaded by backend-software-developer-agent when implementing business logic, designing services, or refactoring code for better maintainability."
user-invocable: false
---

# SOLID Principles — TypeScript Cheat Sheet

## S — Single Responsibility Principle

**Rule:** A class/module should have only one reason to change.

```typescript
// WRONG: UserService handles auth, email, and persistence
class UserService {
  async register(data: CreateUserDto) {
    const hash = await bcrypt.hash(data.password, 10);
    const user = await this.db.user.create({ ...data, password: hash });
    await this.sendWelcomeEmail(user.email);
    return user;
  }
}

// CORRECT: Split into focused services
class UserRepository {
  async create(data: CreateUserDto & { password: string }) {
    return this.db.user.create({ data });
  }
}
class AuthService {
  async hashPassword(plain: string): Promise<string> {
    return bcrypt.hash(plain, 10);
  }
}
class EmailService {
  async sendWelcome(email: string): Promise<void> { /* ... */ }
}
```

## O — Open/Closed Principle

**Rule:** Open for extension, closed for modification. Add behavior without editing existing code.

```typescript
// WRONG: Adding a new discount type means editing this function
function applyDiscount(type: string, price: number): number {
  if (type === "seasonal") return price * 0.9;
  if (type === "loyalty") return price * 0.85;
  // must edit here for every new type
}

// CORRECT: Strategy via interface
interface DiscountStrategy {
  apply(price: number): number;
}
class SeasonalDiscount implements DiscountStrategy {
  apply(price: number) { return price * 0.9; }
}
class LoyaltyDiscount implements DiscountStrategy {
  apply(price: number) { return price * 0.85; }
}
function applyDiscount(strategy: DiscountStrategy, price: number): number {
  return strategy.apply(price);
}
```

## L — Liskov Substitution Principle

**Rule:** Subtypes must be usable wherever their parent type is expected without breaking behavior.

```typescript
// WRONG: ReadOnlyRepo breaks the contract of BaseRepo
class BaseRepo<T> {
  async save(entity: T): Promise<T> { /* ... */ }
}
class ReadOnlyRepo<T> extends BaseRepo<T> {
  async save(): Promise<T> { throw new Error("Read-only!"); }
}

// CORRECT: Separate interfaces
interface Readable<T> {
  findById(id: string): Promise<T | null>;
}
interface Writable<T> extends Readable<T> {
  save(entity: T): Promise<T>;
}
```

## I — Interface Segregation Principle

**Rule:** Clients should not depend on methods they do not use. Prefer small, focused interfaces.

```typescript
// WRONG: Forces every implementation to handle all methods
interface Worker {
  code(): void;
  review(): void;
  manageSprints(): void;
}

// CORRECT: Split by role
interface Coder { code(): void; }
interface Reviewer { review(): void; }
interface Manager { manageSprints(): void; }

class Developer implements Coder, Reviewer {
  code() { /* ... */ }
  review() { /* ... */ }
}
```

## D — Dependency Inversion Principle

**Rule:** High-level modules depend on abstractions, not concrete implementations. Inject dependencies.

```typescript
// WRONG: Direct dependency on concrete implementation
class OrderService {
  private repo = new PostgresOrderRepo();
  async place(order: Order) { return this.repo.save(order); }
}

// CORRECT: Depend on abstraction, inject via constructor
interface OrderRepository {
  save(order: Order): Promise<Order>;
  findById(id: string): Promise<Order | null>;
}

class OrderService {
  constructor(private readonly repo: OrderRepository) {}
  async place(order: Order) { return this.repo.save(order); }
}

// Wire up at composition root
const repo = new PostgresOrderRepo();      // implements OrderRepository
const service = new OrderService(repo);
```

## Quick Reference

| Principle | Key Question |
|---|---|
| SRP | Does this class have more than one reason to change? |
| OCP | Can I add behavior without modifying existing code? |
| LSP | Can I substitute any subtype without surprises? |
| ISP | Am I forcing implementers to stub unused methods? |
| DIP | Am I importing concrete classes in business logic? |

Apply these iteratively during code review — not every file needs all five. Focus on SRP and DIP first; they deliver the most impact in typical Node.js services.

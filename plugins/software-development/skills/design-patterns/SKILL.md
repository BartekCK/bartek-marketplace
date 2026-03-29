---
name: design-patterns
description: "Design patterns reference for TypeScript backend development — GoF patterns, DDD aggregates, and Clean Architecture layers. Loaded by backend-software-developer-agent when designing services, implementing complex business logic, or structuring application architecture."
user-invocable: false
disable-model-invocation: true
---

# Design Patterns — TypeScript Cheat Sheet

## Creational: Factory

**When:** Object creation logic is complex or varies by context.

```typescript
interface Notification { send(to: string, body: string): Promise<void>; }

class EmailNotification implements Notification { /* ... */ }
class SmsNotification implements Notification { /* ... */ }

function createNotification(channel: "email" | "sms"): Notification {
  const map: Record<string, () => Notification> = {
    email: () => new EmailNotification(),
    sms: () => new SmsNotification(),
  };
  return map[channel]();
}
```

## Behavioral: Strategy

**When:** Multiple interchangeable algorithms for the same operation.

```typescript
interface PricingStrategy {
  calculate(base: number, quantity: number): number;
}
class BulkPricing implements PricingStrategy {
  calculate(base: number, qty: number) { return qty > 100 ? base * qty * 0.85 : base * qty; }
}
class OrderService {
  constructor(private pricing: PricingStrategy) {}
  total(base: number, qty: number) { return this.pricing.calculate(base, qty); }
}
```

## Behavioral: Observer (Event Emitter)

**When:** Decouple side effects from core logic.

```typescript
import { EventEmitter } from "node:events";

const bus = new EventEmitter();
bus.on("order.placed", (order: Order) => emailService.sendConfirmation(order));
bus.on("order.placed", (order: Order) => analyticsService.track("purchase", order));

// In OrderService:
async place(order: Order) {
  const saved = await this.repo.save(order);
  bus.emit("order.placed", saved);
  return saved;
}
```

## Structural: Decorator

**When:** Add behavior to a class without modifying it.

```typescript
class CachedUserRepo implements UserRepository {
  constructor(private inner: UserRepository, private cache: Map<string, User>) {}
  async findById(id: string): Promise<User | null> {
    if (this.cache.has(id)) return this.cache.get(id)!;
    const user = await this.inner.findById(id);
    if (user) this.cache.set(id, user);
    return user;
  }
  async save(user: User) { this.cache.delete(user.id); return this.inner.save(user); }
}
```

## Data Access: Repository

**When:** Abstract data access behind a collection-like interface.

```typescript
interface Repository<T, ID = string> {
  findById(id: ID): Promise<T | null>;
  findAll(filter?: Partial<T>): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: ID): Promise<void>;
}
```

## Creational: Singleton (Use With Caution)

Prefer dependency injection over true singletons. In Node.js, module-level instances are effectively singletons already.

```typescript
// Module-scoped instance — simple and testable when injected
export const prisma = new PrismaClient();
```

Avoid `static getInstance()` patterns — they hide dependencies and break testing.

## DDD Building Blocks

### Value Object
Immutable, compared by value, no identity.

```typescript
class Money {
  constructor(readonly amount: number, readonly currency: string) {}
  add(other: Money): Money {
    if (this.currency !== other.currency) throw new Error("Currency mismatch");
    return new Money(this.amount + other.amount, this.currency);
  }
  equals(other: Money) { return this.amount === other.amount && this.currency === other.currency; }
}
```

### Aggregate
Cluster of entities with a single root that guards invariants.

```typescript
class Order {
  private items: OrderItem[] = [];
  addItem(product: Product, qty: number) {
    if (this.status !== "draft") throw new Error("Cannot modify placed order");
    this.items.push(new OrderItem(product.id, product.price, qty));
  }
  get total(): Money { return this.items.reduce((sum, i) => sum.add(i.subtotal), new Money(0, "USD")); }
}
```

### Domain Event
Record what happened; process side effects asynchronously.

```typescript
interface DomainEvent { readonly occurredAt: Date; readonly type: string; }
class OrderPlaced implements DomainEvent {
  readonly type = "order.placed";
  constructor(readonly orderId: string, readonly occurredAt = new Date()) {}
}
```

## Clean Architecture Layers

```
Domain        — Entities, Value Objects, interfaces (zero dependencies)
Application   — Use cases / services (depends on Domain only)
Infrastructure — DB repos, HTTP clients, frameworks (implements Domain interfaces)
Interface      — Controllers, routes, CLI (calls Application services)
```

**Key rule:** Dependencies point inward. Infrastructure implements Domain interfaces; Domain never imports Infrastructure.

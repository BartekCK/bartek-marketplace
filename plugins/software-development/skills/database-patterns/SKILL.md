---
name: database-patterns
description: "Database access patterns for TypeScript/Node.js with Prisma, Drizzle, and Knex. Loaded by backend-software-developer-agent when implementing data access layers, writing queries, or designing database schemas."
user-invocable: false
disable-model-invocation: true
---

# Database Patterns — TypeScript Cheat Sheet

## Repository Pattern with Generics

Abstract data access behind a typed interface. Keeps business logic ORM-agnostic.

```typescript
interface Repository<T, CreateDto, UpdateDto> {
  findById(id: string): Promise<T | null>;
  findMany(filter?: Partial<T>): Promise<T[]>;
  create(data: CreateDto): Promise<T>;
  update(id: string, data: UpdateDto): Promise<T>;
  delete(id: string): Promise<void>;
}
```

## Prisma

### Schema

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  orders    Order[]
  createdAt DateTime @default(now())
}
```

### Typed Repository

```typescript
import { PrismaClient, User, Prisma } from "@prisma/client";

class PrismaUserRepo implements Repository<User, Prisma.UserCreateInput, Prisma.UserUpdateInput> {
  constructor(private prisma: PrismaClient) {}

  findById(id: string) { return this.prisma.user.findUnique({ where: { id } }); }
  findMany() { return this.prisma.user.findMany(); }
  create(data: Prisma.UserCreateInput) { return this.prisma.user.create({ data }); }
  update(id: string, data: Prisma.UserUpdateInput) {
    return this.prisma.user.update({ where: { id }, data });
  }
  async delete(id: string) { await this.prisma.user.delete({ where: { id } }); }
}
```

### N+1 Prevention (Prisma)

```typescript
// WRONG: N+1 — fetches orders per user in a loop
const users = await prisma.user.findMany();
for (const u of users) { u.orders = await prisma.order.findMany({ where: { userId: u.id } }); }

// CORRECT: Eager load with include
const users = await prisma.user.findMany({ include: { orders: true } });
```

## Drizzle

### Schema & Select

```typescript
import { pgTable, uuid, text, timestamp } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: uuid("id").primaryKey().defaultRandom(),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  createdAt: timestamp("created_at").defaultNow(),
});

// Typed query
const result = await db.select().from(users).where(eq(users.email, "a@b.com"));
// result is typeof { id: string; email: string; name: string; createdAt: Date }[]
```

### Joins (N+1 Prevention)

```typescript
const usersWithOrders = await db
  .select()
  .from(users)
  .leftJoin(orders, eq(users.id, orders.userId));
```

## Knex (Typed Query Builder)

```typescript
interface UserRow { id: string; email: string; name: string; created_at: Date; }

const users = await knex<UserRow>("users")
  .where("email", "like", "%@example.com")
  .orderBy("created_at", "desc")
  .limit(20);
```

## Migration Best Practices

1. **One concern per migration** — do not mix schema changes with data backfills.
2. **Always provide a `down`** — reversible migrations save production incidents.
3. **Non-destructive first** — add new column, backfill, switch reads, then drop old column.
4. **Use transactions** for DDL when the database supports it (Postgres does, MySQL does not for DDL).
5. **Name migrations descriptively**: `20240315_add_users_email_index.ts`.

## Connection Pooling

```typescript
// Prisma — set pool in connection string
// postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=10

// Knex
const knex = require("knex")({
  client: "pg",
  connection: { /* ... */ },
  pool: { min: 2, max: 20, idleTimeoutMillis: 30000 },
});
```

**Rules of thumb:**
- Pool size = `(core_count * 2) + spindle_count` (for traditional setups)
- Serverless: use external poolers (PgBouncer, Prisma Accelerate, Neon pooler)
- Always set `pool_timeout` / `idleTimeoutMillis` to avoid leaked connections

## Transactions

```typescript
// Prisma interactive transaction
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.order.create({ data: { ...orderData, userId: user.id } });
});

// Knex transaction
await knex.transaction(async (trx) => {
  const [user] = await trx("users").insert(userData).returning("*");
  await trx("orders").insert({ ...orderData, user_id: user.id });
});
```

## Quick Rules

- Always parameterize queries — never interpolate user input into SQL
- Index foreign keys and columns used in WHERE/ORDER BY
- Use `EXPLAIN ANALYZE` to verify query plans before shipping
- Prefer `select` over `select *` — fetch only needed columns
- Soft-delete (`deleted_at` timestamp) for audit-critical tables

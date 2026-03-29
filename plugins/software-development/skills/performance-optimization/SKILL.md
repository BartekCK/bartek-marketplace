---
name: performance-optimization
description: "Performance optimization patterns for both backend (Node.js) and frontend (React) TypeScript applications. Loaded by backend-software-developer-agent and frontend-software-developer-agent when optimizing application performance."
user-invocable: false
disable-model-invocation: true
---

# Performance Optimization — TypeScript Cheat Sheet

## Backend: Caching Strategies

### In-Memory Cache (Simple TTL)

```typescript
class MemoryCache<T> {
  private store = new Map<string, { value: T; expiry: number }>();

  get(key: string): T | undefined {
    const entry = this.store.get(key);
    if (!entry || Date.now() > entry.expiry) { this.store.delete(key); return undefined; }
    return entry.value;
  }

  set(key: string, value: T, ttlMs: number) {
    this.store.set(key, { value, expiry: Date.now() + ttlMs });
  }
}
```

### Redis Cache (Distributed)

```typescript
import Redis from "ioredis";
const redis = new Redis();

async function cached<T>(key: string, ttl: number, fn: () => Promise<T>): Promise<T> {
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit) as T;
  const result = await fn();
  await redis.setex(key, ttl, JSON.stringify(result));
  return result;
}

// Usage
const user = await cached(`user:${id}`, 300, () => userRepo.findById(id));
```

**Cache invalidation:** Invalidate on write. Use key patterns (`user:*`) with `SCAN` — never `KEYS` in production.

## Backend: Async Patterns

```typescript
// Parallel independent operations
const [user, orders, preferences] = await Promise.all([
  userRepo.findById(id),
  orderRepo.findByUserId(id),
  prefRepo.findByUserId(id),
]);

// Controlled concurrency for batch processing
import pMap from "p-map";
await pMap(items, (item) => processItem(item), { concurrency: 10 });
```

## Backend: N+1 Prevention

Use DataLoader to batch and deduplicate within a single request.

```typescript
import DataLoader from "dataloader";

const userLoader = new DataLoader<string, User>(async (ids) => {
  const users = await db.user.findMany({ where: { id: { in: [...ids] } } });
  const map = new Map(users.map((u) => [u.id, u]));
  return ids.map((id) => map.get(id) ?? new Error(`User ${id} not found`));
});

// Anywhere in request scope — batches automatically
const user = await userLoader.load(userId);
```

## Backend: Connection Pooling

```typescript
// Set pool size based on expected concurrency
// Rule: pool_size = (cpu_cores * 2) + disk_spindles
// Serverless: use external pooler (PgBouncer, Neon pooler)
const pool = new Pool({ max: 20, idleTimeoutMillis: 30000, connectionTimeoutMillis: 5000 });
```

## Backend: Worker Threads

Offload CPU-heavy tasks to avoid blocking the event loop.

```typescript
import { Worker, isMainThread, parentPort, workerData } from "node:worker_threads";

if (isMainThread) {
  function runWorker(data: unknown): Promise<unknown> {
    return new Promise((resolve, reject) => {
      const worker = new Worker(__filename, { workerData: data });
      worker.on("message", resolve);
      worker.on("error", reject);
    });
  }
  const result = await runWorker({ csv: largeData });
} else {
  const parsed = heavyParse(workerData.csv);
  if (!parentPort) throw new Error("No parent port");
  parentPort.postMessage(parsed);
}
```

## Frontend: React Memoization

```typescript
// React.memo — skip re-render when props unchanged
const UserCard = React.memo(({ user }: { user: User }) => (
  <div>{user.name}</div>
));

// useMemo — expensive computation
const sorted = useMemo(() => [...items].sort((a, b) => a.price - b.price), [items]);

// useCallback — stable function reference for child components
const handleClick = useCallback((id: string) => { dispatch(selectItem(id)); }, [dispatch]);
```

**Rule:** Do not memoize everything. Profile first. Memoize when a component re-renders often with the same props or a computation is measurably slow.

## Frontend: Code Splitting & Lazy Loading

```typescript
import { lazy, Suspense } from "react";

const AdminDashboard = lazy(() => import("./pages/AdminDashboard"));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/admin" element={<AdminDashboard />} />
      </Routes>
    </Suspense>
  );
}
```

## Frontend: Image Optimization

```typescript
// Next.js Image (automatic optimization)
import Image from "next/image";
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />

// Native lazy loading
<img src="photo.jpg" alt="Photo" loading="lazy" decoding="async" />
```

## Frontend: Bundle Analysis

```bash
# Vite
npx vite-bundle-visualizer

# Webpack
npx webpack-bundle-analyzer stats.json
```

**Key targets:** Remove duplicate dependencies, tree-shake unused exports, split vendor chunks, lazy-load routes.

## Quick Rules

| Area | Rule |
|------|------|
| Caching | Cache reads at the boundary; invalidate on writes |
| Async | Use `Promise.all` for independent calls; limit concurrency for batches |
| DB | Batch with DataLoader; use `include`/joins; index query columns |
| React | Profile before memoizing; lazy-load below the fold |
| Bundle | Analyze regularly; code-split per route; tree-shake imports |

---
name: logging-observability
description: "Structured logging and observability patterns for TypeScript/Node.js backends. Loaded by backend-software-developer-agent when implementing logging, error tracking, or monitoring in server-side applications."
user-invocable: false
---

# Logging & Observability — TypeScript Cheat Sheet

## Structured Logging with Pino

Pino is the fastest Node.js logger. Always log JSON — never unstructured strings.

```typescript
import pino from "pino";

const logger = pino({
  level: process.env.LOG_LEVEL || "info",
  formatters: {
    level: (label) => ({ level: label }),  // "info" not 30
  },
  redact: ["req.headers.authorization", "body.password"],
});

// Usage — always pass context object first
logger.info({ userId: "abc", action: "login" }, "User logged in");
logger.error({ err, orderId: "xyz" }, "Payment failed");
```

## Log Levels

| Level | When to Use |
|-------|-------------|
| `fatal` | App cannot continue — process will exit |
| `error` | Operation failed, needs attention |
| `warn` | Unexpected but recoverable (deprecated API called, retry succeeded) |
| `info` | Business events (user created, order placed, job completed) |
| `debug` | Developer troubleshooting (query params, intermediate state) |
| `trace` | Extremely verbose (function entry/exit) — never in production |

**Rule:** Production runs at `info`. Use `debug` only when actively investigating.

## Correlation IDs

Attach a unique ID to every request so all logs for one operation are traceable.

```typescript
import { randomUUID } from "node:crypto";
import { AsyncLocalStorage } from "node:async_hooks";

const als = new AsyncLocalStorage<{ requestId: string }>();

// Express middleware
function correlationMiddleware(req: Request, res: Response, next: NextFunction) {
  const requestId = (req.headers["x-request-id"] as string) || randomUUID();
  res.setHeader("x-request-id", requestId);
  als.run({ requestId }, () => next());
}

// Child logger with correlation
function getLogger() {
  const store = als.getStore();
  return store ? logger.child({ requestId: store.requestId }) : logger;
}
```

## Request Context Logging (Express)

```typescript
import pinoHttp from "pino-http";

app.use(pinoHttp({
  logger,
  genReqId: (req) => req.headers["x-request-id"] || randomUUID(),
  serializers: {
    req: (req) => ({ method: req.method, url: req.url, params: req.params }),
    res: (res) => ({ statusCode: res.statusCode }),
  },
  customLogLevel: (_req, res) => (res.statusCode >= 500 ? "error" : "info"),
}));
```

## Error Serialization

Always serialize errors properly — plain `JSON.stringify(err)` returns `{}`.

```typescript
logger.error({ err: pino.stdSerializers.err(error), context: "paymentService" }, "Charge failed");

// Custom serializer for domain errors
function serializeDomainError(err: DomainError) {
  return {
    type: err.constructor.name,
    message: err.message,
    code: err.code,
    stack: err.stack,
    metadata: err.metadata,
  };
}
```

## Winston Alternative

```typescript
import winston from "winston";

const logger = winston.createLogger({
  level: "info",
  format: winston.format.combine(winston.format.timestamp(), winston.format.json()),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "error.log", level: "error" }),
  ],
});
```

## Health Check Endpoint

Every service needs a health endpoint for load balancers and orchestrators.

```typescript
app.get("/health", async (_req, res) => {
  const checks = {
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
    database: await checkDb(),    // "ok" | "degraded" | "down"
    redis: await checkRedis(),
  };
  const healthy = Object.values(checks).every((v) => v !== "down");
  res.status(healthy ? 200 : 503).json({ status: healthy ? "healthy" : "unhealthy", ...checks });
});
```

## Metrics Basics

Use `prom-client` for Prometheus-compatible metrics.

```typescript
import { Counter, Histogram, register } from "prom-client";

const httpRequests = new Counter({
  name: "http_requests_total",
  help: "Total HTTP requests",
  labelNames: ["method", "route", "status"],
});

const httpDuration = new Histogram({
  name: "http_request_duration_seconds",
  help: "HTTP request latency",
  labelNames: ["method", "route"],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
});

// Middleware
app.use((req, res, next) => {
  const end = httpDuration.startTimer({ method: req.method, route: req.route?.path || req.path });
  res.on("finish", () => {
    end();
    httpRequests.inc({ method: req.method, route: req.route?.path || req.path, status: res.statusCode });
  });
  next();
});

app.get("/metrics", async (_req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});
```

## Quick Rules

- Log at service boundaries (incoming request, outgoing call, response)
- Never log sensitive data (tokens, passwords, PII) — use `redact`
- Include `err` key for errors so serializers activate automatically
- Use `debug` for development, strip `trace` from production builds
- Alert on error rate spikes, not individual errors

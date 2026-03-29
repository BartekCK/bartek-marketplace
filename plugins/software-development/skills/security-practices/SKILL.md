---
name: security-practices
description: Security best practices for TypeScript full-stack applications — input validation, authentication patterns, and common vulnerability prevention. Loaded by both backend-software-developer-agent and frontend-software-developer-agent when handling user input, authentication, or data sanitization.
user-invocable: false
disable-model-invocation: true
---

# Security Practices Cheat Sheet

## Input Validation: Parse, Don't Validate

Use Zod at every trust boundary. Validate on entry, use typed data everywhere else.

```typescript
import { z } from "zod";

const CreateUserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(1).max(100).trim(),
  age: z.number().int().min(13).max(150).optional(),
});

type CreateUserInput = z.infer<typeof CreateUserSchema>;

// Express middleware
function validate<T>(schema: z.ZodSchema<T>) {
  return (req: Request, _res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      throw new ValidationError(
        result.error.issues.map((i) => ({ field: i.path.join("."), message: i.message })),
      );
    }
    req.body = result.data; // Typed and sanitized
    next();
  };
}

app.post("/api/users", validate(CreateUserSchema), createUserHandler);
```

## SQL Injection Prevention

NEVER concatenate user input into queries. Always use parameterized queries.

```typescript
// BAD — SQL injection
const users = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

// GOOD — parameterized
const users = await db.query("SELECT * FROM users WHERE email = $1", [email]);

// GOOD — ORM (Prisma)
const user = await prisma.user.findUnique({ where: { email } });
```

## XSS Prevention

React escapes JSX expressions by default. Respect this.

```tsx
// SAFE — React escapes automatically
<p>{userInput}</p>

// DANGEROUS — never use unless absolutely necessary
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// If you must render HTML, sanitize first
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent) }} />
```

**Rules:** Never insert user input into `href` with `javascript:` protocol. Validate URLs.

```typescript
function isSafeUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    return ["http:", "https:"].includes(parsed.protocol);
  } catch {
    return false;
  }
}
```

## CSRF Protection

Use the `csrf-csrf` package. Send token in a header, validate on server.

```typescript
import { doubleCsrf } from "csrf-csrf";

const { doubleCsrfProtection, generateToken } = doubleCsrf({
  getSecret: () => env.CSRF_SECRET,
  cookieName: "__csrf",
  cookieOptions: { httpOnly: true, sameSite: "strict", secure: true },
});

app.use(doubleCsrfProtection);
app.get("/api/csrf-token", (req, res) => res.json({ token: generateToken(req, res) }));
```

## JWT and Authentication

**Never store JWTs in localStorage.** Use httpOnly cookies.

```typescript
import jwt from "jsonwebtoken";

function issueTokens(userId: string, res: Response) {
  const accessToken = jwt.sign({ sub: userId }, env.JWT_SECRET, { expiresIn: "15m" });
  const refreshToken = jwt.sign({ sub: userId, type: "refresh" }, env.JWT_REFRESH_SECRET, { expiresIn: "7d" });

  res.cookie("access_token", accessToken, { httpOnly: true, secure: true, sameSite: "strict", maxAge: 15 * 60 * 1000 });
  res.cookie("refresh_token", refreshToken, { httpOnly: true, secure: true, sameSite: "strict", path: "/api/auth/refresh", maxAge: 7 * 24 * 60 * 60 * 1000 });
}

// Auth middleware
function authenticate(req: Request, _res: Response, next: NextFunction) {
  const token = req.cookies.access_token;
  if (!token) throw new AuthError();
  try {
    const payload = jwt.verify(token, env.JWT_SECRET) as { sub: string };
    req.userId = payload.sub;
    next();
  } catch {
    throw new AuthError("Invalid or expired token");
  }
}
```

## Rate Limiting

```typescript
import rateLimit from "express-rate-limit";

const apiLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100, standardHeaders: true });
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 10 }); // Stricter for auth

app.use("/api/", apiLimiter);
app.use("/api/auth/", authLimiter);
```

## Security Headers with Helmet

```typescript
import helmet from "helmet";
app.use(helmet()); // Sets X-Content-Type-Options, X-Frame-Options, CSP, HSTS, etc.
```

## CORS Configuration

```typescript
import cors from "cors";

app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(",") ?? [],
  credentials: true, // Required for cookies
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
}));
```

## Environment Variables

**Never hardcode secrets.** Validate env vars at startup.

```typescript
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  ALLOWED_ORIGINS: z.string(),
  NODE_ENV: z.enum(["development", "production", "test"]),
});

const env = EnvSchema.parse(process.env); // Fails fast if missing
export default env;
```

## Key Rules

- **Validate all input** at trust boundaries with Zod — never trust client data.
- **Parameterize all queries** — zero exceptions.
- **httpOnly + secure + sameSite cookies** for tokens — never localStorage.
- **Helmet + CORS + rate limiting** on every Express/Fastify app.
- **Fail fast** on missing environment variables at startup.
- **Sanitize HTML** with DOMPurify if you must render user-generated HTML.

---
name: brutal-evaluation
description: "Use this skill when performing code quality evaluations, architecture reviews, or technical assessments that require a structured rating framework. Defines the 1-10 rating scale, 8 evaluation categories with pass/fail criteria, the report template, and scoring rules including weakest-link overall scoring and CRITICAL issue caps."
---

# Brutal Evaluation Framework

## Rating Scale

| Score | Meaning |
|-------|---------|
| 10 | Exceptional — textbook quality, could be used as a teaching example |
| 9 | Excellent — minor stylistic nitpicks only, production-ready with confidence |
| 8 | Good — meets professional standards, no significant issues (MINIMUM PASS) |
| 7 | Acceptable — works but has notable weaknesses that should be addressed |
| 6 | Below average — multiple issues that would cause problems in production |
| 5 | Poor — fundamental problems that need rework |
| 4 | Bad — significant design or implementation flaws |
| 3 | Very bad — barely functional, major rework required |
| 2 | Terrible — more wrong than right |
| 1 | Broken — does not work or is dangerous |

**Minimum PASS threshold: 8/10.** Anything below 8 is a FAIL.

## Evaluation Categories

### 1. Correctness
**8+ requires:** All logic paths produce correct results. Edge cases handled. No off-by-one errors. No silent data corruption. Contracts between functions honored.
**Below 8 signals:** Missing edge cases, incorrect boundary conditions, logic errors under specific inputs, race conditions, incorrect state transitions.

### 2. Error Handling
**8+ requires:** All failure modes identified and handled. Errors propagated meaningfully (not swallowed). User-facing errors are clear. No unhandled promise rejections or uncaught exceptions. Retry/fallback strategies where appropriate.
**Below 8 signals:** Bare catch blocks, swallowed errors, generic error messages, missing validation at system boundaries, no distinction between recoverable and fatal errors.

### 3. Architecture & Design
**8+ requires:** Clear separation of concerns. Appropriate abstractions (not too many, not too few). Dependencies flow in one direction. Easy to extend without modifying existing code. No god objects or god functions.
**Below 8 signals:** Circular dependencies, leaky abstractions, mixed responsibilities, over-engineering for current requirements, tight coupling between unrelated modules.

### 4. Readability & Maintainability
**8+ requires:** Code reads like well-written prose. Naming is precise and consistent. Functions do one thing. Files are focused. A new team member can understand the flow without a walkthrough.
**Below 8 signals:** Cryptic names, functions over 50 lines, deeply nested conditionals, magic numbers/strings, inconsistent conventions within the same codebase.

### 5. Security
**8+ requires:** Input validated at all system boundaries. No injection vulnerabilities (SQL, XSS, command). Secrets not hardcoded. Authentication/authorization checks in place. Principle of least privilege followed.
**Below 8 signals:** Unsanitized user input, hardcoded credentials, missing auth checks, overly permissive CORS/permissions, sensitive data in logs or error messages.

### 6. Performance
**8+ requires:** No unnecessary O(n^2) or worse algorithms. No N+1 queries. Resources cleaned up (connections, file handles, listeners). No blocking operations on hot paths. Appropriate caching where data is reused.
**Below 8 signals:** Unnecessary loops within loops, unbounded memory growth, missing pagination, synchronous I/O on main thread, redundant computations, missing indexes on queried fields.

### 7. Testing
**8+ requires:** Critical paths covered by tests. Tests are deterministic and fast. Test names describe behavior, not implementation. Edge cases tested. Mocks used appropriately (not excessively).
**Below 8 signals:** No tests, flaky tests, tests that test implementation details, missing coverage of error paths, tests that always pass, snapshot tests without assertion context.

### 8. API Design (when applicable)
**8+ requires:** Consistent naming conventions. Predictable behavior across endpoints/methods. Clear input/output contracts. Versioning strategy in place. Errors returned in a consistent format.
**Below 8 signals:** Inconsistent naming, surprising side effects, missing or inconsistent error formats, breaking changes without versioning, ambiguous parameter names.

## Report Template

```
## Quality Evaluation Report

### Scores

| Category | Score | Status |
|----------|-------|--------|
| Correctness | X/10 | PASS/FAIL |
| Error Handling | X/10 | PASS/FAIL |
| Architecture & Design | X/10 | PASS/FAIL |
| Readability & Maintainability | X/10 | PASS/FAIL |
| Security | X/10 | PASS/FAIL |
| Performance | X/10 | PASS/FAIL |
| Testing | X/10 | PASS/FAIL |
| API Design | X/10 | PASS/FAIL/N/A |
| **Overall** | **X/10** | **PASS/FAIL** |

### Issues

#### CRITICAL
- **[Category]** `file:line` — Problem description
  - Best practice: [what should be done, with reference]
  - Required fix: [specific code change needed]

#### MAJOR
- **[Category]** `file:line` — Problem description
  - Best practice: [what should be done, with reference]
  - Required fix: [specific code change needed]

#### MINOR
- **[Category]** `file:line` — Problem description
  - Best practice: [what should be done, with reference]
  - Suggested fix: [specific code change suggested]

### References
1. [Source title](URL) — what it was used for
2. [Source title](URL) — what it was used for
3. [Source title](URL) — what it was used for

### What Was Done Right
- [Genuine positive observation with specifics]
- [Another genuine positive observation]

### Verdict
**PASS/FAIL** — [1-2 sentence summary justifying the overall score]
```

## Scoring Rules

1. **Overall score = weakest category score** (not the average). A chain is only as strong as its weakest link. If Security is 4/10 but everything else is 9/10, the overall score is 4/10.
2. **Any CRITICAL issue caps its category at 5/10 maximum.** A critical security vulnerability means Security cannot score above 5, regardless of how good everything else in that category is.
3. **When in doubt, score lower.** The benefit of the doubt is earned through evidence, not assumed. If you're torn between a 7 and an 8, it's a 7.
4. **N/A categories are excluded** from the overall score calculation. Only score API Design when there is a meaningful API surface to evaluate.
5. **Credit where earned.** The "What Was Done Right" section is mandatory. Even failing code usually has something positive. Be specific — "good naming" is lazy; "consistent use of domain terminology in function names (e.g., `calculatePremium`, `validateClaim`)" is useful.

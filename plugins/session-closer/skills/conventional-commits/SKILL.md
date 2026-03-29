---
name: conventional-commits
description: "Provides the Conventional Commits format specification for git commit messages, including type definitions, formatting rules, and examples. Triggers on committing changes, writing commit messages, 'format my commit', 'how do I write a commit message', 'conventional commits', or 'what commit type should I use'. Also used by the session-closer-agent when committing session documentation."
user-invocable: true
---

# Conventional Commits

Format: `<type>(<optional scope>): <description>`

## Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace — no logic change |
| `refactor` | Code restructuring — no feature or fix |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system or dependencies |
| `ci` | CI/CD configuration |
| `chore` | Maintenance tasks, tooling |
| `revert` | Reverting a previous commit |
| `session` | Session closing — summaries, doc sync *(project-specific, not part of official spec)* |

## Rules

1. **Subject line**: `type(scope): imperative description` — max 72 chars
2. **Scope**: Optional, in parentheses — identifies the area (e.g., `feat(auth):`, `fix(api):`, `session(003):`)
3. **Breaking changes**: Add `!` after type/scope — `feat!: remove legacy API`
4. **Body**: Separated by blank line — explain what and why, not how
5. **Footer**: `BREAKING CHANGE: description` or `Refs: #issue`
6. **Case**: Type and scope are lowercase. Description starts lowercase.
7. **No period** at end of subject line

## Examples

```
feat(auth): add OAuth2 login flow

fix: resolve race condition in queue processor

docs(readme): update installation instructions

refactor(api): extract validation middleware

session(002): plugin split and cleanup
```

## Multi-line Example

```
feat(database): add connection pooling

Introduces configurable connection pool with min/max limits.
Defaults to 5 min, 20 max connections.

Refs: #142
BREAKING CHANGE: DATABASE_URL now requires pool parameters
```

## Session Commits

When committing session memory files or session-closing work:
- Use type `session` with the session number as scope
- Description summarizes the session's main focus
- Body lists what was saved

```
session(007): auth system redesign decisions

- memory/sessions/session_007_auth_redesign.md
- MEMORY.md index updated
```

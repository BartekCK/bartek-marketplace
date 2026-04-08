# software-architect

Framework-agnostic software architect plugin for Claude Code. Intercepts implementation requests ("create", "implement", "build", "add", "refactor", "start", "go") **before any code is written** and runs a rigid research → plan → validate → delegate workflow.

## What it does

1. **Researches** the existing codebase — structure, conventions, tech stack, similar prior implementations
2. **Plans** the architecture — files to touch, layering, data flow, edge cases, risks
3. **Validates** the plan against good practices and project conventions (optionally via `brutal-critic`)
4. **Delegates** implementation to the `frontend-software-developer-agent`, `backend-software-developer-agent`, and `tester-agent` from the `software-development` plugin
5. **Defers** to `nextjs-architect-agent` from the `software-architect-nextjs` plugin if the project is Next.js

The architect is read-mostly: it does **not** write production code. It may write short-lived validation scripts or prototypes into `./tmp/` in the project, and removes them at the end of its work.

## Components

- `agents/software-architect-agent.md` — the architect
- `skills/architecture-planning/SKILL.md` — rigid research → plan → validate → delegate methodology

## Dependencies

- **`software-development` plugin (required)** — provides the `frontend-software-developer-agent`, `backend-software-developer-agent`, and `tester-agent` that this architect delegates implementation to. Without it, there is nothing to delegate to.
- **`software-architect-nextjs` plugin (required for Next.js projects)** — the generic architect defers to `nextjs-architect-agent` on any Next.js project. If not installed, the architect will tell you and offer to fall back to generic planning at reduced fidelity.
- **`brutal-critic` plugin (required for high-risk plans)** — automatically dispatched for plans matching the high-risk criteria (cross-cutting changes ≥3 modules, security-sensitive surfaces, data migrations on live tables, public API changes, performance-critical paths, first use of a new library/pattern). The user can also force brutal-critic validation on any plan.

## Installation

Installed via this marketplace (`.claude-plugin/marketplace.json`).

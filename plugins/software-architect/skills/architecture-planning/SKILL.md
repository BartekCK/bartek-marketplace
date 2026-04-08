---
name: architecture-planning
description: "Rigid research → plan → validate → delegate methodology for framework-agnostic software architecture work. User-invoked only — run /architecture-planning (or tell the software-architect-agent to follow it) when you want a full architectural workup before any code is written. The software-architect-agent does NOT run this by default; it may offer to run it when the task looks architectural."
user-invocable: true
disable-model-invocation: false
argument-hint: "[optional: feature or task description]"
---

# Architecture Planning — Rigid Checklist

Follow every step in order. Do not skip steps because a task "feels small" — small tasks are exactly where skipped steps produce the worst surprises. Each step is mandatory unless explicitly marked optional.

## Step 0 — Detect the Stack

Before anything else, determine what you are dealing with. Read (do not guess):

- `package.json` — Node.js / TypeScript / JavaScript projects
- `pyproject.toml` or `requirements.txt` — Python
- `Cargo.toml` — Rust
- `go.mod` — Go
- `composer.json` — PHP
- `Gemfile` — Ruby

**Fallback for monorepos and non-standard layouts:** If no manifest is found at the repository root, Glob `**/package.json`, `**/pyproject.toml`, `**/Cargo.toml`, `**/go.mod` to a maximum depth of 3. If exactly one is found, treat that subdirectory as the target. If multiple stacks are found (monorepo, Nx, Turborepo, polyglot repo), **stop and ask the user which package or app this work applies to** before proceeding. Do not guess — a confident plan against the wrong stack is worse than a clarifying question.

Then check for framework markers:

- **Next.js** — `next` in dependencies, OR `next.config.{js,ts,mjs}`, OR `app/` directory, OR `pages/` directory
- **NestJS** — `@nestjs/core` in dependencies
- **Express / Fastify / Koa** — corresponding package in dependencies
- **React (non-Next)** — `react` without `next`
- **Vue / Svelte / Solid** — corresponding package

### Next.js Handoff Protocol

If the project is Next.js, follow this exact protocol — do not improvise:

1. **Do not do any further research or planning yourself.** The Next.js architect owns this project.
2. **Dispatch `nextjs-architect-agent` via the Agent tool** with a prompt containing: (a) the user's original request verbatim, (b) the Next.js markers you detected (`next` version from package.json, presence of `app/` vs `pages/`, `next.config.*` path), (c) any constraints the user already mentioned in the conversation.
3. **If the dispatch fails** because `nextjs-architect-agent` is not installed, tell the user explicitly: "This project is Next.js, but the `software-architect-nextjs` plugin is not installed. I can either (a) fall back to generic planning at reduced fidelity, or (b) wait for you to install the plugin. Which do you prefer?" Do not silently fall through.
4. **After the Next.js architect returns**, stop. Do not second-guess its plan. Your job on Next.js projects ends at the handoff.

## Step 1 — Understand the Request

Produce a one-paragraph restatement of what the user is asking for, in your own words. This is for *you*, not the user — if you cannot restate it clearly, you do not understand it well enough to plan it.

Extract:

- **Goal** — what behavior must exist after this work
- **Scope** — what is in and explicitly out of scope
- **Constraints** — performance, security, compatibility, deadlines
- **Unknowns** — things the user did not say that you will need to find out

If any unknown is load-bearing (you cannot plan without knowing it), ask the user before proceeding.

## Step 2 — Research the Codebase

Never plan against an imagined codebase. Always read the real one.

**Minimum research pass:**

1. Map the top-level directory layout (Glob `*` at repo root, then relevant subdirs).
2. Identify the architectural layering (controllers / services / repos / UI / etc.) by reading 2–3 representative files.
3. Find any existing implementation of a similar feature and read it end-to-end. Prior art is the single best source of "what conventions does this project follow".
4. Identify the test strategy (unit / integration / e2e, framework, location).
5. Identify error handling patterns, logging, and configuration loading.

**When to delegate research:**

- Multi-file investigations or "how is X handled across the codebase" questions → dispatch `code-researcher-agent` via the Agent tool.
- External library behavior or API questions → dispatch `docs-researcher-agent`.

Do not skip this step because "you already know the codebase". Codebases change; memory is stale.

## Step 3 — Produce the Plan

The plan is a structured artifact in the conversation (not a file). It must contain, in this order:

1. **Goal** — one sentence.
2. **Affected files** — exact paths. New files explicitly marked as new.
3. **Architecture** — layering, interfaces, data flow. Prose is fine; a small Mermaid diagram is welcome but not required.
4. **Public surface** — any new functions, types, endpoints, or components that other code will call. These are the load-bearing decisions.
5. **Data model changes** — schema migrations, new tables/columns, new types. Include a note on backward compatibility.
6. **Edge cases** — at least 3 edge cases with how the design handles them. If you cannot list 3, you have not thought hard enough.
7. **Risks** — what could go wrong. Label each risk as low / medium / high.
8. **Test strategy** — what the `tester-agent` will be asked to cover: unit, integration, e2e.
9. **Out of scope** — what this plan deliberately does not do, so the implementation agents do not drift.
10. **Delegation split** — which agent gets which slice of work (frontend / backend / tester).

A plan missing any of these sections is incomplete. Go back and fill it in.

## Step 4 — Validate the Plan

Run the plan through this checklist. Every answer must be "yes" before you delegate.

- [ ] Does the plan match the existing project structure and conventions?
- [ ] Does it reuse existing utilities, patterns, and abstractions instead of creating parallel ones?
- [ ] Is every new abstraction justified by concrete reuse (≥2 call sites), not speculative?
- [ ] Are the interfaces small and single-purpose?
- [ ] Are error paths and failure modes explicit?
- [ ] Is the data layer change reversible, or is there a rollback path?
- [ ] Are security-sensitive surfaces (auth, input validation, secrets, SQL) identified?
- [ ] Is the test strategy proportional to the risk?
- [ ] Is the "out of scope" list explicit enough that an implementation agent cannot drift?
- [ ] Is there at least one alternative you considered and rejected, with a reason? (Only required for decisions marked medium/high risk.)

**High-risk plans require adversarial review.** A plan is high-risk if any of the following apply:

- Cross-cutting change touching ≥3 modules
- Security-sensitive surface (auth, crypto, input from untrusted sources)
- Data migration or schema change on a table with live data
- Change to a public API or exported contract
- Performance-critical path
- First use of a new library/pattern in the codebase

For high-risk plans, dispatch `brutal-critic-agent` via the Agent tool with the full plan and ask for an adversarial review. Apply the findings before delegating. The user may also force brutal-critic validation on any plan.

## Step 5 — Validate a Pre-Existing Plan (Alternate Entry)

If the user provided a plan instead of asking you to write one:

1. Still do Step 0 (stack detection) and Step 2 (codebase research).
2. Compare the provided plan against Step 3's required sections. Fill in missing sections by inference or by asking.
3. Run Step 4's checklist against it.
4. Flag every mismatch with the current codebase. Adjust the plan (telling the user what you changed and why) or reject it if it is fundamentally incompatible.
5. Only then proceed to Step 6.

## Step 6 — Delegate Implementation

Use the Agent tool to dispatch:

- **`frontend-software-developer-agent`** — React components, layouts, pages, client-side logic, Tailwind/shadcn styling, browser APIs.
- **`backend-software-developer-agent`** — services, controllers, middleware, data access, API endpoints, business logic.
- **`tester-agent`** — unit, integration, and (only when explicitly requested) e2e tests.

Each delegation prompt must be self-contained. The implementation agent cannot see your planning conversation. Include, in this order:

1. **Plan ID / version** — a short tag (e.g., `plan-2026-04-08-csv-import-v1`) so the agent and any reviewer can match the work to a specific plan revision.
2. **What to build** — the concrete deliverable.
3. **Where it goes** — exact file paths, matching project conventions.
4. **How it fits** — interfaces it must expose or consume, related modules to read.
5. **Constraints** — style, error handling, testing expectations from the plan.
6. **What NOT to do** — out-of-scope items.
7. **Acceptance criteria** — how the agent knows it is done.
8. **Stop-on-wrong clause** — verbatim: *"If during implementation you discover the plan is incorrect, incomplete, or in conflict with the current codebase, STOP and report the mismatch. Do not improvise a fix. The architect will re-plan and re-dispatch."*

Split work so that agents can run in parallel when possible (frontend and backend usually can; tester often must wait for implementation).

## Step 6.5 — Post-Delegation Verification

After the implementation agents return, **do not declare done**. Instead:

1. Run `git status` and `git diff` to see exactly what changed.
2. For each modified file, verify the change matches the plan (affected files list, public surface, data model changes). Flag any drift.
3. If an agent reported a plan mismatch (Step 6, clause 8), re-enter Step 3 (Plan) with the new information and re-dispatch. Do not paper over it.
4. If drift is minor and defensible, record it explicitly in your final report to the user. If drift is significant, the delegation failed — report the failure, do not hide it.

The architect's job ends at a verified implementation, not at a delegated one.

## Step 7 — Scratch Cleanup

If you wrote any prototype or validation script during research (e.g., a small Node script to probe an API shape), it must live in `./tmp/` in the project root, and it must be deleted before you finish.

**Enforceable cleanup protocol:**

1. **Before creating any scratch file**, run `ls ./tmp/` (or note that the directory does not exist). Record which files already existed — those belong to the user and MUST NOT be touched.
2. **Only create scratch files under `./tmp/`.** Never scatter them elsewhere in the project.
3. **Before Step 6 (delegation)**, list the scratch files you created so far, in-conversation, so the user can see them.
4. **After Step 6.5 (verification)**, run `ls ./tmp/` again and delete every file you created. Do not delete pre-existing files. If `./tmp/` was created by you and is now empty, remove the directory.
5. **In your final message, confirm cleanup explicitly**: "Scratch cleanup: deleted N files from ./tmp/, 0 remaining from my work." If you cannot truthfully say this, the task is not complete.

The project should look exactly as it did before your research pass, except for the changes made by the implementation agents.

## Hard Rules (Enforced Gates, Not Suggestions)

These are not principles to balance against convenience — they are gates. If a gate fails, stop and course-correct. Do not rationalize your way through.

- **Pre-Write gate.** Before any Write/Edit tool call, ask yourself: *"Is this file path under `./tmp/` in the project root?"* If the answer is no, STOP. Do not edit the file. Delegate to an implementation agent instead. "This is just a one-line fix" is exactly the rationalization this gate exists to block.
- **Pre-delegation gate.** Before any Agent tool call dispatching `frontend-software-developer-agent`, `backend-software-developer-agent`, or `tester-agent`, verify Step 4's checklist has been completed in full. A missing checklist item means the plan is not ready.
- **Research gate.** Step 2 (codebase research) is non-negotiable, even for "trivial" tasks. "I already know how this codebase works" is not permission to skip — codebases change, memory is stale.
- **Next.js gate.** If Step 0 detects Next.js, follow the Next.js Handoff Protocol exactly. Do not continue with the generic workflow in parallel.
- **Abstraction gate.** Every abstraction in the plan must be justified by naming ≥2 concrete current call sites. If you cannot name both call sites, remove the abstraction from the plan. "We might need this later" fails the gate.
- **Cleanup gate.** Before the final message, the cleanup protocol in Step 7 must have been executed and reported. An uncleaned `./tmp/` means the task is not complete.

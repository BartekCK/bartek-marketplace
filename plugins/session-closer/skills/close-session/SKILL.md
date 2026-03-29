---
name: close-session
description: "Summarizes what changed during a session, writes a session document, and handles smart committing. Triggers on 'close session', 'save session', 'end of session', 'wrap up session', 'summarize session', 'what did we do', 'session summary'."
user-invocable: true
---

# Close Session

Analyze what changed during the session, write a session summary document to `./docs/sessions/`, and help commit properly.

## When Invoked Directly

Run all steps below.

## When Invoked by session-closer-agent

Skip Step 1 — the agent has already gathered context and will provide it. Steps 2-3 still apply.

---

## Step 1: Analyze Changes

Gather context about what happened this session:

1. `git diff --stat` — uncommitted changes
2. `git diff --stat --cached` — staged changes
3. Conversation context — what was discussed and worked on

Combine into a session picture. The conversation context is the primary source — git fills in details.

If the user says "you already know" or similar, synthesize entirely from conversation context and git.

---

## Step 2: Write Session Document

### Determine session number

Scan `./docs/sessions/` for existing session files.

- If directory doesn't exist: create it and start at `001`
- If directory exists: find the highest `NNN` in filenames, use `NNN + 1`

Session files follow the pattern: `session_NNN_short_description.md`

Derive the short description (2-4 words, snake_case) from the session's main focus.

### Write the file

Create `./docs/sessions/session_NNN_description.md`:

```markdown
# Session NNN — YYYY-MM-DD — Short Title

## Summary
Detailed implementation-level summary of what was done. Describes features
added, bugs fixed, refactors applied — at the feature/change level, not
file-level or commit-level. Multiple paragraphs if the session was substantial.

## Decisions
- **[Choice]**: [reasoning]
```

### Content rules

- **Summary is required** — always describe what changed at the implementation level
- **Decisions section is optional** — only include when non-obvious choices were made during the session
- Be specific in the summary: "Implemented JWT auth with refresh token rotation and added login/register endpoints" not "Worked on authentication"
- Decisions must include the "why": "Chose PostgreSQL over SQLite because we need concurrent writes" not "Picked PostgreSQL"

### Example — implementation-heavy session

```markdown
# Session 003 — 2026-03-29 — Auth System Setup

## Summary
Implemented JWT-based authentication with refresh token rotation. Added
login and register endpoints to the API. Set up middleware that validates
tokens on protected routes and returns 401 with clear error messages.
Integrated bcrypt for password hashing with configurable salt rounds.
Updated the user model to include refresh token storage and expiration
tracking.

## Decisions
- **JWT over session cookies**: API-first architecture, need stateless auth for mobile clients
- **Refresh token rotation**: Single-use refresh tokens limit damage from token theft
```

### Example — pure implementation session (no decisions)

```markdown
# Session 004 — 2026-03-29 — Dashboard UI

## Summary
Built the main dashboard page with a stats grid showing active users,
revenue, and error rate. Added a recent activity feed that live-updates
via SSE. Implemented responsive layout that collapses the sidebar on
mobile. Connected all components to the existing API endpoints.
```

---

## Step 3: Smart Commit

Use the `conventional-commits` skill for commit message format reference.

### No uncommitted changes

If `git diff --stat` and `git diff --stat --cached` show nothing: offer to commit only the session document.

```
docs: add session NNN summary — <short description>
```

### Uncommitted changes exist

Analyze the diff to determine if changes span multiple concerns:

**Single concern** — draft one conventional commit message:

1. Present the draft message for user approval
2. Stage specific files (never `git add -A` or `git add .`)
3. Commit after approval

**Multiple concerns** — suggest splitting:

1. Identify the distinct concerns (e.g., "This looks like a feature addition and a bug fix — want me to split them into separate commits?")
2. If user agrees: draft a message for each commit, present all for approval, then commit sequentially — staging only the relevant files for each
3. If user declines: draft a single commit message covering everything

### Commit the session document

**If committing with code changes:** Include the session file in the same commit. The commit message covers the code work; the session file is just another file in the changeset.

**If committing separately** (user preference or no code changes):

```
docs: add session NNN summary — <short description>
```

Use `conventional-commits` skill for format reference. Stage only the session file. Commit after user approval.

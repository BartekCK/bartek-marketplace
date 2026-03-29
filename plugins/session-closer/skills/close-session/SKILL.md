---
name: close-session
description: "Saves session decisions and reasoning to auto memory. Analyzes git history and conversation to capture what was decided and why. Triggers on 'close session', 'save session', 'record session', 'end of session', 'wrap up session', 'log decisions', 'record what we decided', 'save what we learned'."
user-invocable: true
---

# Close Session

Save the key decisions and reasoning from this session to auto memory so future conversations have context.

## When Invoked Directly

Run all steps below.

## When Invoked by session-closer-agent

Skip Steps 1-2 — the agent has already gathered session context and will provide it. Start at Step 3.

---

## Step 1: Assess What Changed

Run these commands to understand the session:

1. `git log --oneline -15` — recent commits
2. `git diff --stat HEAD` — unstaged changes
3. `git diff --stat --cached` — staged changes

Build a picture of what was worked on.

## Step 2: Ask the User

Ask: **"What were the key decisions this session and why?"**

Listen for:
- Architecture or design choices made
- Approaches chosen over alternatives (and why)
- Constraints discovered or trade-offs accepted
- Conventions or patterns established

If the user says "you already know" or similar, synthesize from git history and conversation context.

## Step 3: Determine Session Number

Scan `~/.claude/projects/<project>/memory/sessions/` for existing session files.

- If directory doesn't exist: start at `001`
- If directory exists: find the highest `NNN` in filenames, use `NNN + 1`

Session files follow the pattern: `session_NNN_short_description.md`

Derive the short description (2-4 words, snake_case) from the session's main focus.

## Step 4: Write Session Memory File

All `memory/` paths below are relative to the project's Claude memory directory (`~/.claude/projects/<project>/memory/`).

Create `memory/sessions/session_NNN_description.md` with this format:

```
---
name: Session NNN — Short Description
description: <one-line summary of what was decided, specific enough to judge relevance>
type: project
---

# Session NNN — YYYY-MM-DD — Short Description

## Decisions

- **[Decision 1]**: [What was chosen] — [Why this approach over alternatives]
- **[Decision 2]**: [What was chosen] — [Why]
- ...
```

**Content rules:**
- Only decisions and reasoning — no change lists, no technical debt, no next steps
- Each decision must include the "why" — a decision without reasoning is useless in future context
- Be specific: "Chose PostgreSQL over SQLite because we need concurrent writes" not "Picked a database"
- Omit trivial decisions that won't matter in future conversations

## Step 5: Update MEMORY.md Index

Add a one-liner to `MEMORY.md` (in the same memory directory) under a Sessions section. If `MEMORY.md` does not exist, create it.

```
- [Session NNN: Short Description](sessions/session_NNN_description.md) — one-line hook summarizing key decisions
```

If no Sessions section exists in `MEMORY.md`, create one.

## Step 6: Offer to Commit

Ask: **"Want me to commit the session memory files?"**

- If yes: stage only the memory files and commit using conventional-commits format, e.g., `session(003): capture auth architecture decisions`
- If no: done — files are written but uncommitted

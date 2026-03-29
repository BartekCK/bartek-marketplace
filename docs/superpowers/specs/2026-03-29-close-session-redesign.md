# Close Session Skill Redesign

## Problem

The current close-session skill is focused on saving decisions and reasoning to memory. This misses the primary value: documenting what actually changed during a session at the implementation level. Decisions are useful but secondary — the main output should be a summary of what was implemented/changed, plus smart commit handling.

## Design

### Skill Purpose

Analyze what changed during the session, write a session summary document, and help the user commit properly with intelligent commit splitting.

### Settings (unchanged)

`.claude/session-closer.local.md` with `memory_location` setting:
- `project` (default) — saves to `./docs/sessions/`
- `global` — saves to Claude auto memory

### Flow

#### Step 0: Read Settings
Same as current — parse `.claude/session-closer.local.md` for `memory_location`.

#### Step 1: Analyze Changes
- Check `git diff --stat` for uncommitted changes
- Use conversation context to understand what was worked on
- When invoked by session-closer-agent, skip — context already provided

#### Step 2: Write Session Document

Path: resolved from Step 0 (`./docs/sessions/` or global memory).

Format:
```markdown
# Session NNN — YYYY-MM-DD — Short Title

## Summary
Detailed implementation-level summary of what was done. Describes
features added, bugs fixed, refactors applied — at the feature/change
level, not file-level or commit-level. Multiple paragraphs if the
session was substantial.

## Decisions
- **[Choice]**: [reasoning]
```

Rules:
- Summary is always present — describes what changed at implementation level
- Decisions section is optional — only when non-obvious choices were made
- Session numbering: scan existing files, increment highest NNN
- Global mode: update MEMORY.md index. Project mode: skip index.
- Project mode: no YAML frontmatter. Global mode: include frontmatter (auto memory requirement).

#### Step 3: Smart Commit
- If no uncommitted changes: offer to commit only the session document
- If uncommitted changes exist: analyze the diff for distinct concerns
  - **Single concern** — draft one conventional commit message
  - **Multiple concerns** — suggest splitting (e.g., "This looks like a feat and a fix — want me to split into two commits?")
- Present draft message(s) for user approval before committing
- Stage specific files per commit (never `git add -A`)
- Commit the session document separately as a `docs` commit

### Session Document Examples

**Implementation-heavy session:**
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
- **Refresh token rotation**: Security requirement — single-use refresh tokens limit damage from token theft
```

**Pure implementation session (no decisions):**
```markdown
# Session 004 — 2026-03-29 — Dashboard UI

## Summary
Built the main dashboard page with a stats grid showing active users,
revenue, and error rate. Added a recent activity feed that live-updates
via SSE. Implemented responsive layout that collapses the sidebar on
mobile. Connected all components to the existing API endpoints.
```

### Agent Tweak

In session-closer-agent.md, change the recommendation menu from:
- "Save session to memory" → "Save session summary"

### What Changes vs Current

| Aspect | Current | New |
|--------|---------|-----|
| Focus | Decisions + reasoning | Implementation summary + optional decisions |
| Summary | None | Detailed what-changed at feature level |
| User question | "What were the key decisions?" | Synthesize from conversation context + git |
| Commit handling | Offer to commit memory files only | Smart splitting for all uncommitted work |
| Frontmatter | Always (YAML) | Only in global mode |

### What Stays the Same

- Settings file and `memory_location` config
- Session numbering (`session_NNN_description.md`)
- Storage paths (`./docs/sessions/` or global)
- MEMORY.md index in global mode
- Conventional commits format
- Works standalone (`/close-session`) and delegated by agent

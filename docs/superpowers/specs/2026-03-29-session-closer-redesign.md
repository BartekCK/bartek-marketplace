# Session Closer Plugin Redesign

## Problem

The current session-closer agent is a rigid 5-step pipeline (session summary → README → CLAUDE.md → ai-knowledge-sync → commit) that forces every step. Users don't always want all steps, and the forced commit is unwanted. Session files (`sessions/NNN-*.md`) are frozen knowledge Claude can't recall without manually reading them.

## Design

### Agent: Smart Orchestrator

The session-closer-agent becomes a conversational orchestrator that assesses work and lets the user choose what to do.

**Flow:**

1. Analyze git diff/log since last session (or recent meaningful commits)
2. Ask the user: "What did you work on?" (capture context git can't show)
3. Based on assessment, present recommendations + menu:
   - Recommended actions (with reasoning why each is suggested)
   - Available actions
   - Support for shortcuts ("just commit", "all", etc.)
4. Execute chosen actions by delegating to skills
5. After each action, return to menu unless user says they're done

**Menu format:**

```
Based on your session, I'd recommend:
→ [1] Save session to memory (new architecture decisions detected)
→ [2] Sync AI docs (CLAUDE.md has stale info)
→ [3] Commit changes

What would you like? (or "just commit", "all", etc.)
```

**Smart recommendation logic:**

- Architecture decisions or new patterns detected → recommend close-session
- CLAUDE.md mentions outdated info → recommend ai-knowledge-sync
- Multiple AI tool configs exist → recommend ai-knowledge-sync
- Uncommitted staged changes → recommend commit
- README references outdated features → agent updates README directly (no menu item, agent's judgment call)

**Commit handling:**

The agent drafts a commit message based on its git analysis using conventional-commits format, proposes it to the user for approval/adjustment, then commits.

**Shortcuts:**

The agent understands direct commands without full assessment:
- "just commit" → skip to commit with auto-generated message
- "just sync docs" → skip to ai-knowledge-sync
- "close session" → skip to close-session skill

### Skill: close-session (NEW)

**Type:** User-invocable skill (`/close-session`)

**Purpose:** Save session decisions and reasoning to memory.

**Steps:**

1. Analyze git log/diff to understand what changed
2. Ask user: "What were the key decisions this session and why?" (skipped if agent already gathered this)
3. Determine next session number (scan `memory/sessions/`)
4. Write `memory/sessions/session_NNN_description.md` containing:
   - Session date
   - Decisions made + reasoning (why)
5. Update `MEMORY.md` index with one-liner pointing to the session file
6. Ask: "Want me to commit these changes?"

**When invoked by agent:** Steps 1-2 are skipped (agent passes context).
**When invoked directly:** Does its own assessment.

**Session memory file format:**

```
memory/sessions/session_NNN_description.md
```

Content: decisions + why. No change lists, no technical debt tracking, no next steps. Only what helps future Claude conversations understand past reasoning.

### Skill: ai-knowledge-sync (UPDATED)

Keep existing 8-step audit process. Add:

- **New audit target:** Memory files (`~/.claude/projects/<project>/memory/`) — check if memory entries reference stale files, outdated decisions, or removed features
- **New reference doc:** `references/memory.md` — covers memory file structure, `MEMORY.md` index format, 200-line/25KB limit, topic file pattern
- **CLAUDE.md updates** are owned exclusively by this skill (not the agent, not close-session)

### Skill: conventional-commits (UNCHANGED)

No changes needed.

## Removals

- **Delete all `sessions/` files** from project root — clean break, sessions live in memory now
- **Remove session file creation** from CLAUDE.md conventions
- **Agent prompt** — full rewrite (pipeline → orchestrator)

## Component Ownership

| Concern | Owner |
|---------|-------|
| Session decisions → memory | `close-session` skill |
| CLAUDE.md / AI docs sync | `ai-knowledge-sync` skill |
| README updates | Agent (judgment call) |
| Commit format | `conventional-commits` skill |
| Commit execution | Agent (with user approval) |
| Assessment + menu | Agent |

## File Changes

| Action | Path |
|--------|------|
| Rewrite | `agents/session-closer-agent.md` |
| Create | `skills/close-session/SKILL.md` |
| Update | `skills/ai-knowledge-sync/SKILL.md` |
| Create | `skills/ai-knowledge-sync/references/memory.md` |
| Update | `.claude-plugin/plugin.json` (description) |
| Update | `README.md` |
| Delete | `sessions/` directory (project root) |

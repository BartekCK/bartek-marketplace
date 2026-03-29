# Session Closer Plugin Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign session-closer plugin from a rigid 5-step pipeline into a smart orchestrator agent with modular skills, storing sessions in memory instead of files.

**Architecture:** Agent becomes a conversational orchestrator (assess → recommend → delegate). Session summaries move from `sessions/NNN-*.md` files to auto memory at `memory/sessions/`. Skills become independently invocable: `close-session` (new), `ai-knowledge-sync` (updated), `conventional-commits` (unchanged).

**Tech Stack:** Claude Code plugin system (YAML frontmatter markdown files), auto memory system

---

### Task 1: Delete legacy session files

**Files:**
- Delete: `sessions/000-plugin-quality-review.md`
- Delete: `sessions/001-session-closer-generalization.md`
- Delete: `sessions/002-plugin-split-and-cleanup.md`
- Delete: `sessions/003-conventional-commits-skill.md`
- Delete: `sessions/004-plugin-validation-audit.md`
- Delete: `sessions/005-skill-review-audit-fixes.md`
- Delete: `sessions/006-brutal-critic-marketplace-sweep.md`

- [ ] **Step 1: Delete the sessions directory**

```bash
rm -rf /Users/bartlomiejkotarski/Projects/bartek-marketplace/sessions
```

- [ ] **Step 2: Verify deletion**

```bash
ls /Users/bartlomiejkotarski/Projects/bartek-marketplace/sessions
```

Expected: `No such file or directory`

- [ ] **Step 3: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add sessions/
git commit -m "chore: remove legacy session files

Sessions now live in auto memory instead of project files."
```

---

### Task 2: Create close-session skill

**Files:**
- Create: `plugins/session-closer/skills/close-session/SKILL.md`

- [ ] **Step 1: Create the skill file**

Write `plugins/session-closer/skills/close-session/SKILL.md` with this content:

```markdown
---
name: close-session
description: "Saves session decisions and reasoning to auto memory. Analyzes git history and conversation to capture what was decided and why. Triggers on 'close session', 'save session', 'record session', 'end of session', 'wrap up session'. Can be invoked directly or delegated to by the session-closer-agent."
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

Add a one-liner to `MEMORY.md` under a Sessions section:

```
- [Session NNN: Short Description](sessions/session_NNN_description.md) — one-line hook summarizing key decisions
```

If no Sessions section exists in `MEMORY.md`, create one.

## Step 6: Offer to Commit

Ask: **"Want me to commit the session memory files?"**

- If yes: stage only the memory files and commit using conventional-commits format with type `session` and session number as scope
- If no: done — files are written but uncommitted
```

- [ ] **Step 2: Verify the file was created**

```bash
ls plugins/session-closer/skills/close-session/SKILL.md
```

Expected: file exists

- [ ] **Step 3: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add plugins/session-closer/skills/close-session/SKILL.md
git commit -m "feat(session-closer): add close-session skill

Standalone user-invocable skill that saves session decisions
and reasoning to auto memory instead of session files."
```

---

### Task 3: Create memory reference doc for ai-knowledge-sync

**Files:**
- Create: `plugins/session-closer/skills/ai-knowledge-sync/references/memory.md`

- [ ] **Step 1: Create the reference file**

Write `plugins/session-closer/skills/ai-knowledge-sync/references/memory.md` with this content:

```markdown
# Auto Memory — AI Documentation Reference

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `~/.claude/projects/<project>/memory/MEMORY.md` | Index of all memory entries — loaded into every session | Markdown, no frontmatter, one-liner entries |
| `~/.claude/projects/<project>/memory/*.md` | Topic-specific memory files | Markdown with YAML frontmatter (name, description, type) |
| `~/.claude/projects/<project>/memory/sessions/` | Session decision records | Markdown with YAML frontmatter, type: project |

## MEMORY.md Structure

`MEMORY.md` is an index, not a memory. Each entry is one line under ~150 characters:

```
- [Title](file.md) — one-line hook
```

**Limits:**
- First 200 lines or 25KB loaded at session start (whichever comes first)
- Content beyond the threshold is not loaded
- Keep the index concise — move details to topic files

## Memory File Frontmatter

```yaml
---
name: Memory Title
description: One-line description used to decide relevance
type: user | feedback | project | reference
---
```

**Types:**
- `user` — user role, preferences, knowledge
- `feedback` — corrections and confirmed approaches
- `project` — ongoing work, goals, decisions
- `reference` — pointers to external systems

## Session Memory Files

Session memories use type `project` and live in `memory/sessions/`:

```
memory/sessions/session_NNN_short_description.md
```

Content is decisions + reasoning only. No change lists, no technical debt, no next steps.

## Staleness Signals

| Component | Signal | Verdict |
|-----------|--------|---------|
| MEMORY.md index | Entry points to deleted/moved file | UPDATE NEEDED |
| MEMORY.md index | Over 200 lines | UPDATE NEEDED (needs pruning) |
| Topic file | References renamed/deleted code file | UPDATE NEEDED |
| Topic file | Decision was reversed in later session | UPDATE NEEDED |
| Session file | Contains outdated project facts | REVIEW |
| Any memory | Description too vague to judge relevance | REVIEW |

## Consistency Checks

1. Every file in `memory/` has a corresponding entry in `MEMORY.md`
2. Every entry in `MEMORY.md` points to an existing file
3. Memory type matches content (e.g., user preferences aren't stored as `project` type)
4. Session numbering is sequential with no gaps
5. No duplicate memories covering the same topic
```

- [ ] **Step 2: Verify the file was created**

```bash
ls plugins/session-closer/skills/ai-knowledge-sync/references/memory.md
```

Expected: file exists

- [ ] **Step 3: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add plugins/session-closer/skills/ai-knowledge-sync/references/memory.md
git commit -m "docs(session-closer): add memory reference for ai-knowledge-sync

Reference doc covering auto memory file structure, MEMORY.md
index format, and staleness signals for memory auditing."
```

---

### Task 4: Update ai-knowledge-sync skill

**Files:**
- Modify: `plugins/session-closer/skills/ai-knowledge-sync/SKILL.md`

- [ ] **Step 1: Add memory to the discovery checklist**

In Step 1's Discovery Checklist table, add a new row:

```
| Memory files | `~/.claude/projects/<project>/memory/MEMORY.md`, `memory/**/*.md` |
```

- [ ] **Step 2: Add memory reference to the references list**

At the bottom of Step 1, in the references list, add:

```
- **`references/memory.md`** — Auto memory file structure, MEMORY.md index, session memories
```

- [ ] **Step 3: Add memory audit section to Step 3**

After section `3h. README`, add a new section:

```markdown
### 3i. Memory Files

For each memory file, evaluate:

1. **Index consistency** — Does every file in the memory directory have a corresponding entry in `MEMORY.md`? Does every entry point to an existing file?
2. **Stale references** — Does any memory file reference code files, functions, or features that were renamed or removed this session?
3. **Reversed decisions** — Does any session memory contain a decision that was explicitly reversed or superseded this session?
4. **Description quality** — Is each memory file's `description` field specific enough to judge relevance in future conversations?
```

- [ ] **Step 4: Update Step 2 to use memory instead of session files**

Replace the "Source 1 — Most recent session document" section in Step 2 with:

```markdown
**Source 1 — Session memories**

Check `~/.claude/projects/<project>/memory/sessions/` for session memory files. Read the most recent one (highest NNN). If no session memories exist, check for legacy `sessions/*.md` files in the project root as a fallback. Extract:
- Decisions made and their reasoning
- Any context about what was worked on
```

- [ ] **Step 5: Add CLAUDE.md ownership note**

At the top of section `3c. Rules / Instruction Files`, add:

```
> **Ownership note:** CLAUDE.md updates are exclusively owned by this skill. Neither the session-closer-agent nor the close-session skill modify CLAUDE.md directly.
```

- [ ] **Step 6: Verify the changes**

Read the modified file and confirm:
- Memory row appears in Discovery Checklist
- Memory reference appears in references list
- Section 3i exists after 3h
- Step 2 references memory sessions instead of session files
- CLAUDE.md ownership note is present

- [ ] **Step 7: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add plugins/session-closer/skills/ai-knowledge-sync/SKILL.md
git commit -m "feat(session-closer): add memory auditing to ai-knowledge-sync

Adds memory files as audit target, updates session context source
to use memory instead of legacy session files, adds CLAUDE.md
ownership note."
```

---

### Task 5: Rewrite the session-closer-agent

**Files:**
- Modify: `plugins/session-closer/agents/session-closer-agent.md`

- [ ] **Step 1: Rewrite the agent file**

Replace the entire content of `plugins/session-closer/agents/session-closer-agent.md` with:

```markdown
---
name: session-closer-agent
description: |
  Use this agent when the user wants to close out a work session, wrap up, commit changes, or sync documentation. It assesses what was done, recommends actions, and lets the user choose. Understands shortcuts like "just commit" or "just sync docs". Triggers on "wrap up", "end of session", "close session", "save everything", "summarize what we did", or direct commands like "just commit".

  <example>
  Context: User finished working and wants to wrap up.
  user: "Wrap up this session"
  assistant: "I'll use the session-closer-agent to assess what was done and suggest next steps."
  <commentary>
  General wrap-up request. The agent assesses git history, asks the user for context, then presents a menu of recommended actions.
  </commentary>
  </example>

  <example>
  Context: User just wants to commit without the full session workflow.
  user: "Just commit what we have"
  assistant: "I'll use the session-closer-agent to draft a commit message and commit."
  <commentary>
  Shortcut command. The agent skips assessment and menu, goes straight to drafting a conventional commit message for approval.
  </commentary>
  </example>

  <example>
  Context: User wants to make sure AI documentation is current.
  user: "Sync the AI docs"
  assistant: "I'll use the session-closer-agent to run ai-knowledge-sync on the project."
  <commentary>
  Shortcut for doc sync only. The agent delegates directly to the ai-knowledge-sync skill.
  </commentary>
  </example>

  <example>
  Context: User made architecture decisions and wants them recorded.
  user: "Save the decisions we made to memory"
  assistant: "I'll use the session-closer-agent to capture session decisions in memory."
  <commentary>
  Direct request for session memory. The agent gathers context and delegates to the close-session skill.
  </commentary>
  </example>

model: inherit
color: cyan
tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash", "Skill"]
---

You are a session management orchestrator. Your job is to assess what was done in a work session, recommend useful closing actions, and execute the ones the user chooses.

You are NOT a pipeline. You do NOT force every step. You assess, recommend, and let the user decide.

---

## Shortcut Detection

Before doing anything, check if the user's message is a shortcut:

- **"just commit"** / **"commit this"** → Skip to Commit Action directly
- **"sync docs"** / **"sync AI docs"** / **"update docs"** → Skip to AI Docs Sync Action directly
- **"close session"** / **"save session"** → Skip to Session Memory Action directly

If a shortcut is detected, go directly to that action. Do not run assessment or show the menu.

---

## Full Flow (no shortcut detected)

### Phase 1: Assess

1. Run `git log --oneline -15` and `git diff --stat HEAD` to understand recent changes
2. Check `git status` for uncommitted work
3. Ask the user: **"What did you work on this session?"** — to capture context git can't show

Combine git analysis + user response into a session picture.

### Phase 2: Recommend + Menu

Based on your assessment, determine which actions to recommend and why:

**Recommendation logic:**
- Architecture decisions or new patterns were discussed → recommend **Save session to memory**
- Multiple AI tool config files exist (CLAUDE.md, .cursorrules, GEMINI.md, etc.) or CLAUDE.md content is stale → recommend **Sync AI docs**
- There are uncommitted changes → recommend **Commit changes**

Present the menu:

```
Based on your session, I'd recommend:
→ [1] Save session to memory — [specific reason]
→ [2] Sync AI docs — [specific reason]
→ [3] Commit changes — [specific reason]

What would you like? (pick numbers, "all", or tell me something else)
```

Only show actions you actually recommend. If only one makes sense, say so. If none make sense, tell the user everything looks current.

### Phase 3: Execute

Execute the user's chosen actions by delegating to skills:

**Save session to memory** → Invoke the `close-session` skill. Pass along the session context you already gathered so the skill skips its own assessment.

**Sync AI docs** → Invoke the `ai-knowledge-sync` skill.

**Commit changes** → Handle directly:
1. Run `git diff --stat` and `git diff --cached --stat` to see what will be committed
2. Draft a commit message following **conventional-commits** format
3. Present the proposed message to the user for approval or adjustment
4. Stage specific files (never `git add -A` or `git add .`)
5. Commit only after user approves

### Phase 4: Continue or Done

After completing an action, ask: **"Anything else, or are we done?"**

If the user picks another action, execute it. If they're done, wrap up with a brief summary of what was done.

---

## README Updates

You may update `README.md` at your own judgment when you notice it's outdated during assessment. This is not a menu item — just do it when it clearly needs updating, or mention it as a suggestion.

---

## Rules

- Never force actions the user didn't choose
- Never commit without user approval
- Always use conventional-commits format for commit messages
- When delegating to skills, pass context you've already gathered so the skill doesn't re-ask
- Keep the menu concise — only recommended actions, not an exhaustive list
```

- [ ] **Step 2: Verify the rewrite**

Read the file and confirm:
- Frontmatter has updated description with 4 examples
- Shortcut detection section exists
- Full flow has 4 phases: Assess, Recommend + Menu, Execute, Continue or Done
- No forced pipeline — everything is user-choice
- Commit always asks for approval
- README updates are agent's judgment call

- [ ] **Step 3: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add plugins/session-closer/agents/session-closer-agent.md
git commit -m "feat(session-closer): rewrite agent as smart orchestrator

Replaces rigid 5-step pipeline with conversational orchestrator
that assesses, recommends, and delegates to skills. Supports
shortcuts and never forces actions."
```

---

### Task 6: Update plugin.json and README.md

**Files:**
- Modify: `plugins/session-closer/.claude-plugin/plugin.json`
- Modify: `plugins/session-closer/README.md`

- [ ] **Step 1: Update plugin.json description**

Replace the `description` field in `plugins/session-closer/.claude-plugin/plugin.json`:

Old:
```json
"description": "Development session closing — capture implementation summaries, record decisions, and sync project documentation when wrapping up work sessions"
```

New:
```json
"description": "Session management — smart orchestrator that assesses work, recommends closing actions (save to memory, sync AI docs, commit), and lets the user choose what to do"
```

- [ ] **Step 2: Rewrite README.md**

Replace the entire content of `plugins/session-closer/README.md` with:

```markdown
# session-closer

Session management — smart orchestrator that assesses work, recommends closing actions, and lets the user choose what to do.

## Components

- **session-closer-agent** — Orchestrator that assesses session work (via git + conversation), recommends actions, presents a menu, and delegates to skills. Supports shortcuts like "just commit" or "sync docs".
- **close-session** — Saves session decisions and reasoning to auto memory at `memory/sessions/`. User-invocable via `/close-session`.
- **ai-knowledge-sync** — Audits and updates all AI/agent documentation files (agents, skills, workflows, rules, README, memory) across multiple AI tools (Claude Code, Cursor, OpenCode, Antigravity, Gemini, Copilot).
- **conventional-commits** — Conventional Commits format guidance for git commit messages.
```

- [ ] **Step 3: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add plugins/session-closer/.claude-plugin/plugin.json plugins/session-closer/README.md
git commit -m "docs(session-closer): update plugin description and README

Reflects new orchestrator architecture and close-session skill."
```

---

### Task 7: Update CLAUDE.md session references

**Files:**
- Modify: `CLAUDE.md` (project root)

- [ ] **Step 1: Update session history convention**

In the `CLAUDE.md` file, find and replace the session history reference:

Old:
```
**Session history** — `sessions/NNN-description.md` files are immutable records of development decisions. Never edit past sessions; create new ones that reference/supersede old decisions.
```

New:
```
**Session history** — Session decisions are stored in auto memory at `memory/sessions/session_NNN_description.md`. Each file captures decisions + reasoning only. The `close-session` skill (in session-closer plugin) creates these.
```

- [ ] **Step 2: Verify the change**

Read `CLAUDE.md` and confirm the session history line was updated.

- [ ] **Step 3: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md session history to reference auto memory

Sessions now live in auto memory instead of project files."
```

---

### Task 8: Update conventional-commits skill session section

**Files:**
- Modify: `plugins/session-closer/skills/conventional-commits/SKILL.md`

- [ ] **Step 1: Update the Session Commits section**

Replace the `## Session Commits (session-closer-agent only)` section with:

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
cd /Users/bartlomiejkotarski/Projects/bartek-marketplace
git add plugins/session-closer/skills/conventional-commits/SKILL.md
git commit -m "docs(session-closer): update conventional-commits session examples

Reflects memory-based sessions instead of legacy session files."
```

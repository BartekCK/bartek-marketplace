# architecture-software

Development session management plugin for software projects.

## What It Does

Automates end-of-session housekeeping for software development projects. After finishing a coding or design session, trigger the agent to:

1. Generate a comprehensive implementation summary (what was built, decisions made, problems solved)
2. Update the project README with current state
3. Sync CLAUDE.md with architectural decisions, technical debt, and phase progress

## Agents

### `dev-session-closer-agent`

Trigger at the end of any development session to document progress and keep project files in sync.

**Trigger phrases:**
- "Close the session and save everything"
- "Wrap up this session"
- "End of session"
- "Summarize what we implemented today"
- "Make sure CLAUDE.md reflects the decisions we made"

**What it saves:**
- `session-log.md` — appends a new dated implementation summary (what was built, decisions, problems solved, next steps)
- `README.md` — overwrites with current project state and architecture
- `CLAUDE.md` — syncs phase progress, architectural decisions, patterns, and technical debt (only if present)

## Installation

Add to your `.claude-plugin/marketplace.json`:

```json
{
  "name": "architecture-software",
  "source": "./plugins/architecture-software",
  "description": "Development session management — capture implementation summaries, record architectural decisions, update README, and sync CLAUDE.md"
}
```

## Usage

Use this plugin in any software project directory. Works with or without an existing `CLAUDE.md` — the agent adapts to what's present.

Typical project structure:
```
your-project/
├── CLAUDE.md          (optional — synced if present)
├── README.md          (updated each session)
├── session-log.md     (running log of implementation sessions)
└── src/               (your source code)
```

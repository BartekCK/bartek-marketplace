# architecture-software

Software development toolkit plugin for Claude Code. Provides four specialized agents and four supporting skills for common development workflows.

## Agents

### `session-closer-agent`

Closes out a work session by capturing what was accomplished, syncing project documentation, and recording decisions.

**When it triggers:**
- "Close the session and save everything"
- "Wrap up this session"
- "End of session"
- "Summarize what we did today"

**What it updates:**
- `sessions/NNN-description.md` -- creates a numbered session document
- `README.md` -- overwrites with current project state
- `CLAUDE.md` -- syncs architectural decisions (only if file exists)
- Plugin components -- audits and updates plugin files (only if `.claude-plugin/plugin.json` exists)

### `code-researcher-agent`

Investigates a codebase to find patterns, locate references, discover where code lives, and provide placement guidance for new code.

**When it triggers:**
- "Find where we handle authentication errors in the codebase"
- "Where should I put the new UserNotification service?"
- "Check if something similar to a rate limiter already exists"
- "Look at UserController.ts and see how we should implement the new endpoint"

**Does NOT trigger for:** General "grep for X" requests -- those use the built-in Grep tool directly. This agent is for multi-step research requiring synthesis.

### `database-agent`

Connects to databases, inspects schemas, executes queries, and helps developers understand project data. Supports PostgreSQL, MySQL, SQLite, and MongoDB.

**When it triggers:**
- "What tables do we have in the database?"
- "Find all users who signed up in the last 30 days"
- "How do I connect to our database?"
- "Show me sample records from the DB"

**Requires:** A database connection. Set `DATABASE_URL` in your environment before starting Claude Code for full MCP support, or the agent falls back to native CLI tools.

### `documentation-researcher-agent`

Looks up external library documentation, API references, and method signatures from the internet.

**When it triggers:**
- "How do I use the debounce function from lodash?"
- "Show me the docs for the Zod schema library"
- "I'm getting TypeError: axios.create is not a function"
- When another agent encounters an unfamiliar external API

## Skills

### `code-search`

Reference material for modern CLI search tools (ripgrep, fd, fzf). Loaded automatically when search workflows are discussed. Provides command patterns and recipes.

### `db-connect`

Fallback workflow for establishing a database connection mid-session when `DATABASE_URL` was not set at startup. Used internally by the database-agent.

### `web-docs-search`

Search strategies and source hierarchies for finding external documentation. Used internally by the documentation-researcher-agent.

### `session-plugin-sync`

Audit workflow for syncing plugin component files after implementation work. Used internally by the session-closer-agent on plugin projects only.

## Prerequisites

- **Database features:** Set `DATABASE_URL` in your environment before starting Claude Code. The `.mcp.json` configures dbhub for automatic database access.
- **Code search features:** Install `ripgrep`, `fd`, and optionally `fzf` for full search capability.

## Installation

Add to your `.claude-plugin/marketplace.json`:

```json
{
  "name": "architecture-software",
  "source": "./plugins/architecture-software",
  "description": "Software development toolkit -- session closing, codebase research, database inspection, documentation lookup"
}
```

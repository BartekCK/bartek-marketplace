# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin marketplace (`websoftware-tools`) containing 9 independent plugins. Each plugin provides agents, skills, and optional MCP server configs. The marketplace manifest is `.claude-plugin/marketplace.json`.

## Architecture

**Marketplace** — `.claude-plugin/marketplace.json` is the central registry. Each entry points to a plugin directory via `source: "./plugins/<name>"`.

**Plugin structure** — Every plugin follows the same layout:
```
plugins/<name>/
  .claude-plugin/plugin.json   # Manifest (name, version, description, author)
  README.md
  agents/<name>-agent.md       # Agent definition with YAML frontmatter + system prompt
  skills/<skill-name>/SKILL.md # Skill with frontmatter + optional references/ subdir
  .mcp.json                    # Optional MCP server config (browser-manager, database, my-serena, software-architect-nextjs, software-development)
```

**Session history** — Session decisions are stored in auto memory at `memory/sessions/session_NNN_description.md`. Each file captures decisions + reasoning only. The `close-session` skill (in session-closer plugin) creates these.

## Plugin Conventions

- Agent descriptions use YAML block scalar (`description: |`) with `<example>` blocks containing `<commentary>` for trigger disambiguation
- **Skill descriptions must use single-line quoted strings** — NOT `description: |` block scalars, which are broken in Claude Code's skill indexer (bug #12971, session 006)
- Internal/reference skills (loaded by agents, not users) use `user-invocable: false`
- Skills exclusively used by one agent use `disable-model-invocation: true`
- Agent-exclusive skills require BOTH `user-invocable: false` AND `disable-model-invocation: true` (session 006)
- Read-only agents get explicit tool restrictions in their `tools` array (principle of least privilege)
- Plugin `.mcp.json` uses bare server name at root level (e.g. `{"dbhub": {...}}`), NOT wrapped in `"mcpServers"`
- Agent `.md` files reference skills by name, not by file path — directory moves don't require internal edits

## Validation

Run the `plugin-validator` agent on any plugin after changes. Run `brutal-critic-agent` for quality evaluation. There are no automated tests.

## Key Design Decisions

- One concern per plugin — each plugin has a single focused purpose (decided in session 002, superseding session 000's bundle approach)
- MCP configs belong exclusively in the plugin that uses them (e.g. dbhub only in `database` plugin)
- Use `git mv` when reorganizing plugin files to preserve history
- **Agents delegate to skills for format/convention guidance (DRY)** — agents reference skills by name rather than duplicating content inline (session 003)
- **Project-specific convention types must be labeled** — custom types like `session` in conventional-commits are explicitly marked as non-standard (session 003)
- **Commit messages follow Conventional Commits** — use the `conventional-commits` skill in `session-closer` plugin for format reference (session 003)
- **Skill description block scalars are broken** — Claude Code indexer bug #12971 prevents `description: |` from working; use single-line quoted strings instead (session 006)
- **Quality threshold is 8/10** — all plugins must score at least 8/10 on brutal-critic evaluation (session 006)
- **MongoDB is not supported by dbhub** — MariaDB and SQL Server are the correct additional databases (session 006)

## Known Issues / Technical Debt

- `database/db-connect`: `export DATABASE_URL` does not persist across Bash tool calls — Step 4 now instructs embedding DSN directly in each command (session 005, workaround session 006)
- Post-fix brutal-critic re-evaluation pending — all 9 plugins need re-run to confirm 8/10 pass (session 006)

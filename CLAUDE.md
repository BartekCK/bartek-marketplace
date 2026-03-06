# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin marketplace (`websoftware-tools`) containing 7 independent plugins. Each plugin provides agents, skills, and optional MCP server configs. The marketplace manifest is `.claude-plugin/marketplace.json`.

## Architecture

**Marketplace** — `.claude-plugin/marketplace.json` is the central registry. Each entry points to a plugin directory via `source: "./plugins/<name>"`.

**Plugin structure** — Every plugin follows the same layout:
```
plugins/<name>/
  .claude-plugin/plugin.json   # Manifest (name, version, description, author)
  README.md
  agents/<name>-agent.md       # Agent definition with YAML frontmatter + system prompt
  skills/<skill-name>/SKILL.md # Skill with frontmatter + optional references/ subdir
  .mcp.json                    # Optional MCP server config (only browser-manager, database, my-serena have this)
```

**Session history** — `sessions/NNN-description.md` files are immutable records of development decisions. Never edit past sessions; create new ones that reference/supersede old decisions.

## Plugin Conventions

- Agent descriptions use YAML block scalar (`description: |`) with `<example>` blocks containing `<commentary>` for trigger disambiguation
- Internal/reference skills (loaded by agents, not users) use `user-invocable: false`
- Skills exclusively used by one agent use `disable-model-invocation: true`
- Read-only agents get explicit tool restrictions in their `tools` array
- Plugin `.mcp.json` uses bare server name at root level (e.g. `{"dbhub": {...}}`), NOT wrapped in `"mcpServers"`
- Agent `.md` files reference skills by name, not by file path — directory moves don't require internal edits

## Validation

Run the `plugin-validator` agent on any plugin after changes. Run `brutal-critic-agent` for quality evaluation. There are no automated tests.

## Key Design Decisions

- One concern per plugin — each plugin has a single focused purpose (decided in session 002, superseding session 000's bundle approach)
- MCP configs belong exclusively in the plugin that uses them (e.g. dbhub only in `database` plugin)
- Use `git mv` when reorganizing plugin files to preserve history

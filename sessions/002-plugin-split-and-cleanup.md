# Session 002 — 2026-03-06 — Plugin Split and Cleanup

## What Was Implemented

- Split `architecture-software` plugin into 4 independent plugins: `session-closer`, `code-researcher`, `database`, `docs-researcher`
- Created new plugin directories under `plugins/` for each of the 4 plugins
- Used `git mv` to move all agents, skills, and `.mcp.json` files from `plugins/architecture-software/` to the appropriate new plugin directories, preserving full git history
- Created new `plugin.json` and `README.md` for each of the 4 new plugins
- Removed the old `plugins/architecture-software/` directory entirely (including its `.claude-plugin/plugin.json` and `README.md`)
- Updated `.claude-plugin/marketplace.json`: removed `my-superpowers` external plugin entry, replaced single `architecture-software` entry with 4 new plugin entries
- Cleaned up `~/.claude/plugins/installed_plugins.json` to remove stale `architecture-software` and `my-superpowers` references
- Cleaned up stale cache directories
- Ran `plugin-validator` on all 4 new plugins — all passed with 0 critical issues and 0 warnings

## Architectural Decisions

- **One concern per plugin** *(supersedes session 000 decision to keep single bundle)*: Split the monolithic `architecture-software` plugin into 4 single-responsibility plugins — each plugin now has a clear, focused purpose rather than bundling unrelated capabilities together. Session 000 decided against splitting, but session 001 identified the "identity crisis" as the root problem and recommended the split
- **`.mcp.json` moved exclusively to `database` plugin**: The dbhub MCP config was previously loaded for all `architecture-software` users even if they only needed code search or docs lookup — now only `database` plugin users load dbhub, resolving prior tech debt
- **`git mv` for all moves**: Preserves git blame and history tracking across the reorganization, rather than delete-and-recreate which would lose attribution
- **Agent `.md` files left unedited**: Agents reference skills by name (not by file path), so no internal edits were needed after the directory restructure
- **`my-superpowers` removed from marketplace**: External plugin entry removed from `marketplace.json` — no longer offered for installation
- **Session 000 docs left unchanged**: Historical session documents reference the old `architecture-software` plugin name — left as-is since session records are immutable history

## Problems Solved

- **Monolithic plugin bundled unrelated concerns**: `architecture-software` contained session closing, code research, database tooling, and docs research in one plugin — split into 4 focused plugins so users install only what they need
- **dbhub MCP loaded unnecessarily**: All `architecture-software` users loaded the dbhub MCP server regardless of whether they needed database access — resolved by isolating `.mcp.json` in the `database` plugin only
- **Stale installed-plugins references**: After removing `architecture-software` and `my-superpowers`, their entries lingered in `~/.claude/plugins/installed_plugins.json` — manually cleaned up
- **Stale cache directories**: Old cache artifacts from removed plugins cleaned up to avoid confusion

## Patterns & Conventions Established

- Single-responsibility plugin design: each plugin should have one clear purpose with its own `plugin.json`, `README.md`, agents, and skills
- MCP configurations (`.mcp.json`) belong in the plugin that actually uses the MCP server, not in a shared parent plugin
- Use `git mv` when reorganizing plugin files to preserve history
- Session documents are immutable records — do not retroactively edit them when names or structures change

## Technical Debt

- Session 000 still references the old `architecture-software` plugin name throughout — acceptable as historical record but could confuse someone reading linearly
- No automated tests exist for any of the 4 new plugins (carried forward from session 000)
- Each new plugin has its own `README.md` and `plugin.json` that were created fresh — may need refinement as the plugins evolve

## Test Coverage

- All 4 new plugins passed `plugin-validator` with 0 critical issues and 0 warnings
- Manual verification that `git mv` preserved file contents correctly
- No automated test suite exists for plugin functionality

## Next Steps

- Install and verify each of the 4 new plugins independently in a clean Claude Code session
- Test that `database` plugin correctly loads dbhub MCP and the other 3 plugins do not attempt to load it
- Add automated validation to CI/CD if one exists
- Address remaining technical debt from session 000 (zero automated tests, conditional MCP loading within database plugin)
- Consider creating evaluation scenarios for each plugin in isolation

## Open Questions

- Should the 4 new plugins share any common skills or utilities, or remain fully independent?
- Is there a mechanism for plugin dependencies (e.g., `session-closer` using `code-researcher` during discovery) or should each be self-contained?
- Should `my-superpowers` removal be documented anywhere beyond this session record and the marketplace diff?

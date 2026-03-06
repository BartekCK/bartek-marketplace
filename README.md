# claude-plugins

A marketplace of Claude Code plugins for development tooling, research, and session management.

## Current State

The marketplace contains 7 plugins, each with a focused single-responsibility scope. The former monolithic `architecture-software` plugin has been split into 4 independent plugins (`session-closer`, `code-researcher`, `database`, `docs-researcher`). All plugins pass validation. The external `my-superpowers` plugin has been removed from the marketplace.

## What Was Done This Session

- Created `conventional-commits` skill in the `session-closer` plugin for git commit message format guidance
- Updated `session-closer-agent` to delegate commit format to the skill (DRY — no inline duplication)
- Validated with skill-reviewer and brutal-critic; resolved all findings
- Marked the `session` commit type as project-specific in the skill
- Added legacy format note for backward compatibility with earlier sessions

## Plugins

| Plugin | Description |
|--------|-------------|
| `browser-manager` | Browser automation via Chrome DevTools MCP |
| `my-serena` | Semantic coding tools via MCP |
| `session-closer` | Session closing — capture summaries, decisions, and sync docs |
| `code-researcher` | Codebase research and pattern discovery |
| `database` | Database inspection and querying via dbhub MCP |
| `docs-researcher` | External documentation lookup |
| `brutal-critic` | Code quality evaluator with PASS/FAIL reports |

## Architecture

- **Plugin per concern**: Each plugin is self-contained with its own agents, skills, and optional MCP config
- **Marketplace manifest**: `.claude-plugin/marketplace.json` lists all available plugins with source paths
- **Session history**: `sessions/` directory holds immutable session records documenting development decisions

## Project Structure

```
.claude-plugin/
  marketplace.json          # Plugin registry
plugins/
  browser-manager/
  brutal-critic/
  code-researcher/
  database/
  docs-researcher/
  my-serena/
  session-closer/
sessions/
  000-plugin-quality-review.md
  001-session-closer-generalization.md
  002-plugin-split-and-cleanup.md
  003-conventional-commits-skill.md
```

## Getting Started

1. Clone this repository
2. Use the Claude Code plugin installer to install plugins from the marketplace
3. Each plugin directory contains its own `README.md` with usage details

## Next Steps

- Use the conventional-commits skill in practice to validate end-to-end workflow
- Install and verify each plugin independently in a clean Claude Code session
- Test that `database` plugin correctly loads dbhub MCP in isolation
- Add automated validation or CI checks
- Consider shared convention skills for other plugins

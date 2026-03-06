# claude-plugins

A marketplace of Claude Code plugins for development tooling, research, and session management.

## Current State

The marketplace contains 8 plugins, all registered in the marketplace manifest. Session 006 ran a full brutal-critic evaluation across all 8 plugins and applied fixes to bring them up to the 8/10 quality threshold. All major factual errors, DRY violations, deprecated tooling, and convention violations have been resolved. The remaining technical debt is minor (db-connect DSN persistence workaround).

## What Was Done This Session

- Ran brutal-critic-agent on all 8 plugins; only 1/8 passed initially
- Applied fixes across 23 files (141 insertions, 242 deletions) to resolve all findings
- Removed false MongoDB references from database plugin, added MariaDB and SQL Server
- Fixed Claude Code skill indexer bug workaround: converted block scalar descriptions to single-line strings
- Eliminated DRY violations, added least-privilege tools arrays, fixed deprecated/incorrect tooling

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
| `software-development` | Frontend/backend development agents with best-practice skills |

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
  software-development/
sessions/
  000-plugin-quality-review.md
  001-session-closer-generalization.md
  002-plugin-split-and-cleanup.md
  003-conventional-commits-skill.md
  004-plugin-validation-audit.md
  005-skill-review-audit-fixes.md
  006-brutal-critic-marketplace-sweep.md
```

## Getting Started

1. Clone this repository
2. Use the Claude Code plugin installer to install plugins from the marketplace
3. Each plugin directory contains its own `README.md` with usage details

## Next Steps

- Re-run brutal-critic-agent on all 8 plugins to verify they now pass 8/10
- Add automated marketplace-wide validation or CI checks
- Install and verify each plugin independently in a clean Claude Code session
- Consider a marketplace-level quality gate that blocks sub-8/10 plugins

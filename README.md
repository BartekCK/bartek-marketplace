# claude-plugins

A marketplace of Claude Code plugins for development tooling, research, and session management.

## Current State

The marketplace contains 8 plugins, all registered in the marketplace manifest, each with valid structure and metadata. Session 005 completed a full skill-level review across all 7 plugins with skills and fixed all critical issues from the prior validation audit. Remaining technical debt is minor: a functional bug in `db-connect`, a DRY violation in `docs-researcher`, and optional description rewrites.

## What Was Done This Session

- Ran skill-reviewer agent on all 7 plugins, producing Needs Improvement / Pass ratings
- Fixed all 3 critical issues from session 004 (marketplace registration, missing README, missing version)
- Added `disable-model-invocation: true` to 12 agent-exclusive skills across 4 plugins
- Corrected `brutal-evaluation` frontmatter (added `user-invocable: false` + `disable-model-invocation: true`)

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
```

## Getting Started

1. Clone this repository
2. Use the Claude Code plugin installer to install plugins from the marketplace
3. Each plugin directory contains its own `README.md` with usage details

## Next Steps

- Fix `db-connect` skill's `export DATABASE_URL` persistence bug
- Refactor `docs-researcher` agent to delegate to skill content (DRY violation)
- Resolve `parentPort!` non-null assertion contradiction in `performance-optimization`
- Add automated marketplace-wide validation or CI checks
- Install and verify each plugin independently in a clean Claude Code session

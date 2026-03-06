# claude-plugins

A marketplace of Claude Code plugins for development tooling, research, and session management.

## Current State

The marketplace contains 8 plugins, each with a focused single-responsibility scope. A full validation audit (session 004) found all plugins structurally sound with no broken manifests. Three critical issues remain: `software-development` is not yet registered in `marketplace.json`, `my-serena` is missing a README, and `session-closer` has an uncommitted skill rename. Several plugins need minor metadata fixes (`user-invocable`, `disable-model-invocation` flags).

## What Was Done This Session

- Ran `plugin-validator` agent against all 8 marketplace plugins
- Identified 3 critical issues and multiple minor warnings across the plugin set
- Confirmed all plugin manifests are valid JSON with correct structure
- Documented common pattern of missing `user-invocable`/`disable-model-invocation` metadata on internal skills
- No code changes made — read-only validation audit

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

- Fix 3 critical issues: register `software-development` in marketplace, add `my-serena` README, commit `session-closer` skill rename
- Add missing `user-invocable: false` and `disable-model-invocation: true` flags to internal skills
- Add automated marketplace-wide validation or CI checks
- Install and verify each plugin independently in a clean Claude Code session

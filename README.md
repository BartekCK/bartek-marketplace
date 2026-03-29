# claude-plugins

A marketplace of Claude Code plugins for development tooling, research, and session management.

## Current State

The marketplace contains 9 plugins, all registered in the marketplace manifest. All plugins target the 8/10 quality threshold enforced by the brutal-critic plugin. The remaining technical debt is minor (db-connect DSN persistence workaround).

## Plugins

| Plugin | Description |
|--------|-------------|
| `browser-manager` | Browser automation via Chrome DevTools MCP |
| `my-serena` | Semantic coding tools via MCP |
| `session-closer` | Session closing — capture summaries, decisions, and sync docs |
| `code-researcher` | Codebase research and pattern discovery |
| `database` | Database inspection and querying via dbhub MCP |
| `docs-researcher` | External documentation lookup |
| `brutal-critic` | Uncompromising quality evaluator for code and text with PASS/FAIL reports |
| `software-development` | Frontend/backend development agents with best-practice skills |
| `software-architect-nextjs` | Next.js architect agent with MCP-powered docs and implementation delegation |

## Architecture

- **Plugin per concern**: Each plugin is self-contained with its own agents, skills, and optional MCP config
- **Marketplace manifest**: `.claude-plugin/marketplace.json` lists all available plugins with source paths
- **Session history**: Session decisions are stored in auto memory via the `close-session` skill

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
  software-architect-nextjs/
```

## Getting Started

1. Clone this repository
2. Use the Claude Code plugin installer to install plugins from the marketplace
3. Each plugin directory contains its own `README.md` with usage details

## Next Steps

- Re-run brutal-critic-agent on all 9 plugins to verify they pass 8/10
- Add automated marketplace-wide validation or CI checks
- Install and verify each plugin independently in a clean Claude Code session
- Consider a marketplace-level quality gate that blocks sub-8/10 plugins

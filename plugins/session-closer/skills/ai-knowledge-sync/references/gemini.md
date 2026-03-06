# Gemini — AI Documentation Reference

**Note:** There are two distinct Gemini products with different config approaches. Gemini CLI uses `GEMINI.md` at the project root. Gemini Code Assist (IDE extension) uses the `.gemini/` directory for agents and additional configuration. Verify which product applies to the project.

## File Locations

| File | Product | Purpose | Format |
|------|---------|---------|--------|
| `GEMINI.md` | Gemini CLI | Project-level instructions | Markdown |
| `.gemini/config.json` | Both | Project configuration | JSON |
| `.gemini/instructions.md` | Both | Alternative location for project instructions | Markdown |
| `.gemini/agents/*.md` | Gemini Code Assist | Custom agent definitions | Markdown |

## GEMINI.md

The primary instruction file, placed at the project root. Loaded into Gemini CLI conversations automatically.

```markdown
# Project Instructions

## Overview
[What the project does and its current state]

## Architecture
[Key structural decisions, tech stack, directory layout]

## Code Style
[Conventions, naming, patterns to follow]

## Build & Test
```bash
npm run build
npm test
npm run lint
```

## Important Notes
[Things Gemini should always keep in mind — constraints, gotchas, known issues]
```

## Configuration

`.gemini/config.json` for project settings:

```json
{
  "model": "gemini-2.5-pro",
  "instructions_file": "GEMINI.md",
  "context": {
    "include": ["src/**/*.ts", "docs/**/*.md"],
    "exclude": ["node_modules", "dist"]
  }
}
```

## Agent Definitions

For Gemini Code Assist, custom agents in `.gemini/agents/`:

```markdown
# Agent Name

## Role
[What this agent specializes in]

## Context
[What files and knowledge this agent needs]

## Instructions
[Specific behavioral instructions]
```

## Consistency Checks

1. **GEMINI.md matches project state** — Architecture, commands, and conventions reflect reality
2. **Config paths valid** — Include/exclude patterns reference directories that exist
3. **Consistent with other AI docs** — If CLAUDE.md or .cursorrules exist, terminology and facts align
4. **Build commands work** — Commands listed in GEMINI.md are current and functional
5. **No stale architecture** — If directories were reorganized this session, GEMINI.md reflects the new structure

## Key Differences from Other Tools

- Gemini CLI uses `GEMINI.md` at root (similar to `CLAUDE.md`) — the simplest setup of any AI tool
- Gemini Code Assist adds agent capabilities via `.gemini/agents/` — more similar to Claude Code plugins
- Configuration is simpler than Claude Code or Antigravity — fewer abstractions
- Context management relies more on include/exclude patterns than per-agent tool restrictions

**Verify against current docs before use** — Gemini's tooling is evolving rapidly and file locations may change between versions.

# OpenCode — AI Documentation Reference

**Note:** OpenCode is a newer tool with limited public documentation. File locations and formats described here are based on available information — verify against current OpenCode docs before relying on specific paths or config schemas.

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `AGENTS.md` | Agent instructions and configuration | Markdown |
| `.opencode/config.json` | Project configuration | JSON |
| `.opencode/agents/*.md` | Custom agent definitions | Markdown |
| `.opencode/instructions.md` | Project-level instructions | Markdown |

## Configuration

OpenCode uses a JSON config file for project settings:

```json
{
  "model": "claude-sonnet-4-20250514",
  "provider": "anthropic",
  "agents": {
    "custom-agent": {
      "instructions": ".opencode/agents/custom-agent.md",
      "model": "claude-sonnet-4-20250514"
    }
  }
}
```

## Instructions File

`.opencode/instructions.md` is the equivalent of CLAUDE.md — project-level context loaded into every conversation.

```markdown
# Project Instructions

## Overview
[What the project does]

## Architecture
[Key structural decisions]

## Conventions
[Coding standards and patterns]

## Commands
[How to build, test, deploy]
```

## Agent Definitions

Custom agents in `.opencode/agents/` follow a simple markdown format:

```markdown
# Agent Name

## Role
[What this agent does]

## Instructions
[Detailed behavior instructions]

## Tools
[Which tools the agent can use]
```

## AGENTS.md

Top-level file documenting all available agents and their purposes. Acts as a registry and quick reference.

## Consistency Checks

1. **Config references valid files** — Agent paths in `config.json` point to files that exist
2. **AGENTS.md matches actual agents** — All agents in `.opencode/agents/` are documented in AGENTS.md
3. **Instructions match project state** — `.opencode/instructions.md` reflects current architecture and conventions
4. **Model IDs are current** — Referenced models still exist and haven't been deprecated

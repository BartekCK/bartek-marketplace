# Claude Code — AI Documentation Reference

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `CLAUDE.md` | Project-level instructions loaded into every conversation | Markdown with sections for architecture, conventions, commands |
| `.claude-plugin/plugin.json` | Plugin manifest — name, version, description, author | JSON |
| `agents/*.md` | Agent definitions with YAML frontmatter + system prompt body | Markdown with YAML frontmatter |
| `skills/*/SKILL.md` | Skill definitions with YAML frontmatter + instruction body | Markdown with YAML frontmatter |
| `commands/*.md` | Slash commands with YAML frontmatter + prompt body | Markdown with YAML frontmatter |
| `.mcp.json` | MCP server configuration | JSON — bare server name at root level, NOT wrapped in `"mcpServers"` |

## Agent Frontmatter Fields

```yaml
---
name: agent-name
description: |
  When to use this agent. Include <example> blocks with <commentary> for disambiguation.
model: inherit  # or specific model ID
color: cyan     # terminal color
tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
---
```

Key conventions:
- Descriptions use YAML block scalar (`description: |`) with `<example>` blocks
- The `tools` array restricts which tools the agent can use — missing tools cause runtime failures
- Read-only agents get explicit tool restrictions (e.g., only `["Read", "Glob", "Grep"]`)

## Skill Frontmatter Fields

```yaml
---
name: skill-name
description: When to use this skill. Include trigger phrases and contexts.
user-invocable: true         # false for internal/reference skills loaded by agents
disable-model-invocation: true  # true if exclusively used by one agent
---
```

Key conventions:
- Internal skills (loaded by agents, not users) use `user-invocable: false`
- Skills exclusively used by one agent use `disable-model-invocation: true`
- Skills can have `references/` subdirectory for detailed docs loaded on demand

> **Known bug (#12971):** Skill `description` fields must use single-line quoted strings.
> Block scalar (`description: |`) breaks Claude Code's skill indexer. Agent descriptions
> are not affected and should continue using block scalars with `<example>` blocks.

## CLAUDE.md Structure

Typical sections:
- What the project is
- Architecture overview
- Key commands (build, test, lint)
- Conventions and patterns
- Key design decisions
- Known issues / technical debt

CLAUDE.md is always loaded into context, so keep it concise. Detailed reference material belongs in skill reference files, not CLAUDE.md.

## .mcp.json Format

```json
{
  "server-name": {
    "command": "npx",
    "args": ["-y", "@some/mcp-server"],
    "env": {
      "API_KEY": "${API_KEY}"
    }
  }
}
```

Note: bare server name at root level — NOT wrapped in `"mcpServers"`.

## Staleness Signals

| Component | Signal | Verdict |
|-----------|--------|---------|
| Agent | Description omits new capability | UPDATE NEEDED |
| Agent | Tools list missing required tool | UPDATE NEEDED |
| Agent | References renamed/deleted skill | UPDATE NEEDED |
| Skill | Trigger phrases miss new use case | UPDATE NEEDED |
| Skill | References point to moved/deleted file | UPDATE NEEDED |
| Command | References changed file paths or APIs | UPDATE NEEDED |
| MCP | Env var renamed but config uses old name | UPDATE NEEDED |
| Manifest | Description doesn't cover new component | UPDATE NEEDED |
| Any | Component created this session | CURRENT |
| Any | Change is ambiguous or unstable | REVIEW |

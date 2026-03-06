# Antigravity — AI Documentation Reference

**Note:** Antigravity has limited public documentation. File locations and config formats described here may change between versions — verify against current Antigravity docs before relying on specific paths or schemas.

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `.antigravity/config.yaml` | Project configuration and agent settings | YAML |
| `.antigravity/instructions.md` | Project-level AI instructions | Markdown |
| `.antigravity/agents/*.md` | Custom agent definitions | Markdown |
| `.antigravity/workflows/*.yaml` | Workflow definitions | YAML |
| `.antigravity/rules/*.md` | Rule sets for code generation | Markdown |

## Configuration

Antigravity uses YAML for project configuration:

```yaml
project:
  name: my-project
  description: "Project description"

agents:
  code-reviewer:
    instructions: agents/code-reviewer.md
    model: claude-sonnet-4-20250514

workflows:
  pr-review:
    file: workflows/pr-review.yaml

rules:
  - rules/typescript.md
  - rules/testing.md
```

## Instructions File

`.antigravity/instructions.md` provides project-wide context:

```markdown
# Project Context

## What This Is
[Project description]

## Stack
[Technologies used]

## Patterns
[Key architectural patterns]

## Rules
[Project-specific rules and constraints]
```

## Agent Definitions

Agents in `.antigravity/agents/`:

```markdown
# Code Reviewer

## Purpose
Review pull requests for quality, security, and consistency.

## Behavior
- Check for OWASP top 10 vulnerabilities
- Verify test coverage for new code
- Flag style violations

## Output Format
Use structured review comments with severity levels.
```

## Workflow Definitions

Workflows in `.antigravity/workflows/`:

```yaml
name: pr-review
trigger: pull_request
steps:
  - agent: code-reviewer
    action: review
  - agent: test-checker
    action: verify-coverage
```

## Rules Files

Rules in `.antigravity/rules/` define code generation constraints:

```markdown
# TypeScript Rules

- Use strict mode
- No `any` types — use `unknown` and narrow
- Prefer `interface` over `type` for object shapes
- All exported functions must have JSDoc
```

## Consistency Checks

1. **Config paths resolve** — All file references in `config.yaml` point to existing files
2. **Workflow agents exist** — Agents referenced in workflows are defined in `agents/`
3. **Rules don't contradict** — Multiple rule files don't give opposing guidance
4. **Instructions match reality** — Stack and patterns described match the actual codebase

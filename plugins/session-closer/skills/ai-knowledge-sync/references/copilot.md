# GitHub Copilot — AI Documentation Reference

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `.github/copilot-instructions.md` | Repository-level instructions for GitHub Copilot | Markdown |
| `.github/copilot-instructions/*.md` | Modular instruction files (newer feature) | Markdown |

## .github/copilot-instructions.md

The primary instruction file for GitHub Copilot. Loaded into Copilot Chat conversations when working in the repository.

```markdown
# Project Instructions

## Overview
[What the project does]

## Code Style
- Use TypeScript with strict mode
- Prefer functional components
- Use named exports

## Architecture
- src/components/ — React components
- src/services/ — Business logic
- src/utils/ — Shared utilities

## Testing
- Use Jest for unit tests
- Use React Testing Library for component tests
- Every new function needs a corresponding test

## Conventions
- File names: kebab-case
- Component names: PascalCase
- Always handle errors explicitly
```

## Key Characteristics

- Single markdown file — no frontmatter, no YAML config, just plain markdown
- Loaded automatically by GitHub Copilot when present in the repository
- Applies to Copilot Chat, Copilot Edits, and inline completions (scope varies by feature)
- No glob patterns or conditional loading — the entire file is always included
- Maximum recommended size: keep concise, as it counts against context window

## Consistency Checks

1. **Matches project reality** — Do file paths, conventions, and architecture match the actual codebase?
2. **Consistent with other AI docs** — If CLAUDE.md, .cursorrules, or GEMINI.md exist, do all files describe the same architecture and conventions?
3. **No stale patterns** — If conventions changed this session, update the instructions
4. **No contradictory guidance** — Instructions should not conflict with .editorconfig, ESLint rules, or other tooling config

## Common Pitfalls

- Copilot instructions are simpler than other tools — no agent definitions, no workflows, no conditional rules
- Overly long instruction files reduce effectiveness as they compete for context window space
- Unlike Cursor's .mdc files, there is no way to scope instructions to specific file types

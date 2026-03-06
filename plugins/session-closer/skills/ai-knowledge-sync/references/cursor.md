# Cursor — AI Documentation Reference

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `.cursorrules` | Project-level AI instructions loaded into every Cursor chat | Plain text or Markdown |
| `.cursor/rules/*.mdc` | Modular rule files (Cursor 0.45+) | MDC format with frontmatter |
| `.cursorignore` | Files to exclude from Cursor indexing | Gitignore syntax |

## .cursorrules Format (Legacy)

**Note:** `.cursorrules` is the legacy format. Cursor has been steering users toward `.cursor/rules/*.mdc` since Cursor 0.45+. New projects should use the modular format. Existing `.cursorrules` files still work but may be deprecated in future versions.

A single file at the project root. Contains instructions that Cursor loads into every AI conversation. No formal schema — plain text or markdown.

```markdown
# Project Rules

You are working on [project description].

## Code Style
- Use TypeScript strict mode
- Prefer functional components with hooks
- Use named exports

## Architecture
- src/components/ — React components
- src/hooks/ — Custom hooks
- src/utils/ — Utility functions

## Conventions
- File names: kebab-case
- Component names: PascalCase
- Always add unit tests for new functions
```

## .cursor/rules/*.mdc Format (Modular Rules)

Cursor 0.45+ supports splitting rules into multiple files with frontmatter:

```markdown
---
description: Rules for React components
globs: src/components/**/*.tsx
alwaysApply: false
---

- Use functional components with TypeScript
- Props interface must be exported
- Each component gets its own directory with index.tsx and styles.ts
```

Frontmatter fields:
- `description` — When this rule applies (used by Cursor to decide relevance)
- `globs` — File patterns that auto-trigger this rule
- `alwaysApply` — If true, always included regardless of context

## Consistency Checks

When auditing `.cursorrules` or `.cursor/rules/`:

1. **Matches project reality** — Do file paths, conventions, and architecture descriptions match the actual codebase?
2. **Consistent with other AI docs** — If CLAUDE.md exists, do both files describe the same architecture and conventions?
3. **No stale patterns** — If a convention was changed this session (e.g., switched from class components to functional), update the rules.
4. **Glob patterns valid** — For `.mdc` files, verify that `globs` patterns match files that actually exist.

## Common Pitfalls

- `.cursorrules` gets very long over time — prefer modular `.cursor/rules/*.mdc` for large projects
- Rules that reference specific file paths break when files are moved
- Duplicating instructions between `.cursorrules` and `.cursor/rules/` creates contradictions

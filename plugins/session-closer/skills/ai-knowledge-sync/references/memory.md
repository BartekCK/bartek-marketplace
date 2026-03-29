# Auto Memory — AI Documentation Reference

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `~/.claude/projects/<project>/memory/MEMORY.md` | Index of all memory entries — loaded into every session | Markdown, no frontmatter, one-liner entries |
| `~/.claude/projects/<project>/memory/*.md` | Topic-specific memory files | Markdown with YAML frontmatter (name, description, type) |
| `~/.claude/projects/<project>/memory/sessions/` | Session decision records | Markdown with YAML frontmatter, type: project |

## MEMORY.md Structure

`MEMORY.md` is an index, not a memory. Each entry is one line under ~150 characters:

```
- [Title](file.md) — one-line hook
```

**Limits:**
- First 200 lines or 25KB loaded at session start (whichever comes first)
- Content beyond the threshold is not loaded
- Keep the index concise — move details to topic files

## Memory File Frontmatter

```yaml
---
name: Memory Title
description: One-line description used to decide relevance
type: user | feedback | project | reference
---
```

**Types:**
- `user` — user role, preferences, knowledge
- `feedback` — corrections and confirmed approaches
- `project` — ongoing work, goals, decisions
- `reference` — pointers to external systems

## Session Memory Files

Session memories use type `project` and live in `memory/sessions/`:

```
memory/sessions/session_NNN_short_description.md
```

Content is decisions + reasoning only. No change lists, no technical debt, no next steps.

## Staleness Signals

| Component | Signal | Verdict |
|-----------|--------|---------|
| MEMORY.md index | Entry points to deleted/moved file | UPDATE NEEDED |
| MEMORY.md index | Over 200 lines | UPDATE NEEDED (needs pruning) |
| Topic file | References renamed/deleted code file | UPDATE NEEDED |
| Topic file | Decision was reversed in later session | UPDATE NEEDED |
| Session file | Contains outdated project facts | REVIEW |
| Any memory | Description too vague to judge relevance | REVIEW |

## Consistency Checks

1. Every file in `memory/` has a corresponding entry in `MEMORY.md`
2. Every entry in `MEMORY.md` points to an existing file
3. Memory type matches content (e.g., user preferences aren't stored as `project` type)
4. Session numbering is sequential with no gaps
5. No duplicate memories covering the same topic

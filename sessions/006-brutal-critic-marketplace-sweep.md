# Session 006 — 2026-03-06 — Brutal Critic Marketplace Sweep

## What Was Accomplished

- Ran `brutal-critic-agent` on all 8 plugins in parallel, producing scored evaluation reports
- Only 1 plugin (code-researcher, 8/10) passed the 8/10 threshold; 7 plugins failed
- Applied targeted fixes across 23 files (141 insertions, 242 deletions) to bring all plugins up to standard
- Fixed factual errors in `database` plugin: removed false MongoDB references, added actually-supported MariaDB and SQL Server
- Fixed broken `description: |` YAML block scalars in `session-closer` skills — Claude Code indexer bug #12971 causes block scalars to fail skill discovery
- Removed scope creep from `code-researcher` (zoxide/yazi references)
- Fixed deprecated/incorrect tooling in `software-development`: csurf removal, shadcn CLI name, MCP config, Vitest v2+ syntax, Zod-validated env
- Eliminated DRY violations in `docs-researcher` (agent duplicated skill content) and `brutal-critic` (agent duplicated scoring rules)
- Added principle-of-least-privilege `tools` arrays to `brutal-critic` and `browser-manager` agents
- Added `user-invocable: false` and `disable-model-invocation: true` to both `my-serena` skills
- Updated CLAUDE.md to reflect resolved known issues

## Decisions Made

- **YAML block scalars banned for skill descriptions** — `description: |` is broken in Claude Code's skill indexer (bug #12971); single-line quoted strings are required for reliable discovery
- **MongoDB removed from database plugin** — dbhub does not support MongoDB; MariaDB and SQL Server were the actually missing databases
- **Agent-exclusive skills require both flags** — `user-invocable: false` AND `disable-model-invocation: true` must both be present on skills that only an agent should load
- **Read-only agents must have explicit tools arrays** — principle of least privilege; agents that only inspect should not have access to write tools
- **All plugins should pass 8/10 minimum** — this is the quality threshold enforced by brutal-critic-agent

## Problems Solved

- **False database documentation** — database plugin listed MongoDB as supported when dbhub has no MongoDB driver; replaced with MariaDB and SQL Server which are actually supported
- **Skill discovery failures** — session-closer skills using `description: |` block scalar were not being discovered by Claude Code; converted to single-line quoted strings
- **Deprecated security advice** — software-development plugin recommended csurf (deprecated, CVE-laden); removed entirely
- **Incorrect CLI commands** — browser-manager had `npm install -g npx` (npx ships with npm); software-development had wrong shadcn CLI name and removed ESLint `--ext` flag
- **Non-null assertion contradicting strict typing rules** — performance-optimization had `parentPort!` while teaching strict typing; replaced with Zod-validated env pattern
- **DRY violations in agents** — docs-researcher and brutal-critic agents duplicated content from their skills; refactored to delegate

## Patterns & Conventions Established

- **Skill descriptions must use single-line quoted strings** — not YAML block scalars, due to Claude Code indexer bug
- **Quality threshold: 8/10** — all plugins must score at least 8/10 on brutal-critic evaluation
- **Agent-exclusive skill flags** — both `user-invocable: false` and `disable-model-invocation: true` required
- **Least-privilege tools arrays** — read-only agents declare explicit tool lists

## Technical Debt & Open Items

- `database/db-connect`: `export DATABASE_URL` does not persist across Bash tool calls — Step 4 now instructs embedding DSN directly in each command (noted in session 005, workaround applied)
- Post-fix re-evaluation not performed — all 8 plugins should be re-run through brutal-critic to confirm they now pass 8/10

## Next Steps

- Re-run brutal-critic-agent on all 8 plugins to verify all now pass 8/10
- Add automated marketplace-wide validation or CI checks
- Install and verify each plugin independently in a clean Claude Code session
- Consider a marketplace-level quality gate that blocks registration of plugins scoring below 8/10

## Open Questions

- Should the YAML block scalar ban for skill descriptions be reported upstream as a bug, or is it already tracked as #12971?
- Is there a way to automate the brutal-critic sweep as a pre-commit or CI check?

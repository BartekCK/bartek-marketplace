# session-closer

Session management — smart orchestrator that assesses work, recommends closing actions, and lets the user choose what to do.

## Components

- **session-closer-agent** — Orchestrator that assesses session work (via git + conversation), recommends actions, presents a menu, and delegates to skills. Supports shortcuts like "just commit" or "sync docs".
- **close-session** — Saves session decisions and reasoning to auto memory at `memory/sessions/`. User-invocable via `/close-session`.
- **ai-knowledge-sync** — Audits and updates all AI/agent documentation files (agents, skills, workflows, rules, README, memory) across multiple AI tools (Claude Code, Cursor, OpenCode, Antigravity, Gemini, Copilot).
- **conventional-commits** — Conventional Commits format guidance for git commit messages.

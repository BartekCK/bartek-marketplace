# session-closer

Session management — smart orchestrator that assesses work, recommends closing actions, and lets the user choose what to do.

## Components

- **session-closer-agent** — Orchestrator that assesses session work (via git + conversation), recommends actions, presents a menu, and delegates to skills. Supports shortcuts like "just commit" or "sync docs".
- **close-session** — Summarizes what changed during a session, writes a session document to `./docs/sessions/`, and handles smart committing. User-invocable via `/close-session`.
- **ai-knowledge-sync** — Audits and updates all AI/agent documentation files (agents, skills, workflows, rules, README, memory) across multiple AI tools (Claude Code, Cursor, OpenCode, Antigravity, Gemini, Copilot).
- **conventional-commits** — Conventional Commits format guidance for git commit messages.

## How It Works

Run the session-closer agent when you're done working. It checks git history and asks what you worked on, then recommends relevant actions: writing a session summary, syncing AI documentation, or committing changes. You pick what to do — it never forces a pipeline.

Supports shortcuts: say "just commit" or "sync docs" to skip straight to a specific action.

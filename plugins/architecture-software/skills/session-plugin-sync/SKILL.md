---
name: session-plugin-sync
description: Audits and updates plugin component files (agents, skills, commands, .mcp.json, plugin.json) after implementation work. Used internally by the dev-session-closer-agent on plugin projects. Also triggers on "sync the plugin", "update plugin components", or "check if plugin files are current".
disable-model-invocation: true
---

# Session Plugin Sync

## Overview

Plugin component files drift from the actual implementation after every development session. Agents describe stale tools, skills reference files that no longer exist, and the manifest no longer reflects the plugin's scope. Audit all plugin component files against session changes and apply targeted updates.

**Core principle:** Only update what the session actually changed. Leave everything else intact.

---

## Step 1: Detect Plugin Project

Glob for `.claude-plugin/plugin.json`:

```
Glob: .claude-plugin/plugin.json
```

- If **found**: continue to Step 2.
- If **not found**: this is not a plugin project — exit the skill immediately. Report "skipped — not a plugin project".

Do not proceed further if the manifest is absent. All subsequent steps assume a plugin project.

---

## Step 2: Discover All Component Files

Glob for each component type from the project root:

| Component | Glob Pattern |
|-----------|-------------|
| Agents | `agents/*.md` |
| Skills | `skills/*/SKILL.md` |
| Commands | `commands/*.md` |
| MCP config | `.mcp.json` |
| Manifest | `.claude-plugin/plugin.json` |

Build a complete inventory before proceeding. Record which categories have files and which are empty. An empty category is not an error — record it and move on.

---

## Step 3: Load Session Context

Load two sources of session context before beginning any audit:

**Source 1 — Most recent session document**

Glob for `sessions/*.md` and read the highest-numbered file (e.g., `sessions/003-feature-name.md`). If no `sessions/` directory exists, check for a legacy `session-log.md` file instead. Extract:
- Features or components implemented this session
- Files created, moved, or deleted
- API, tool, or environment variable changes
- New agents, skills, or commands added
- Any explicitly noted deprecations or removals

**Source 2 — Git diff**

Run `git diff --stat HEAD~1 HEAD` (or `git diff --stat HEAD` for unstaged changes). Extract:
- Which files changed
- Whether any component files themselves appear in the diff (a component file in the diff is strong signal it was intentionally updated)

Combine both into a session change summary. This summary is the ground truth used throughout the audit. If the session log and git diff contradict each other, prefer the session log — it reflects decisions made, not just file changes.

---

## Step 4: Audit Each Component

Audit every file in the inventory against the session change summary. For each file, determine one of three verdicts. Apply the per-component staleness signals from `references/component-audit-rules.md` and the verdict decision framework from `references/update-criteria.md`.

### Agents (`agents/*.md`)

For each agent file, evaluate three areas:

1. **Description accuracy** — Does the frontmatter `description` field accurately describe what the agent does after this session? Check whether new capabilities were added or existing responsibilities changed. If the agent now invokes a new skill or performs a new task type not mentioned in the description, flag it.

2. **Tools list** — Does the `tools:` frontmatter array include all tools the agent now requires? If the session introduced new Bash commands, file writes, or glob usage in the agent's workflow, the tools list must reflect this. Missing tools cause runtime failures; extra tools are not harmful but flag for `REVIEW` if clearly obsolete.

3. **Skill references** — Does the agent's system prompt reference any skills by name? If those skills were renamed, created, or significantly restructured this session, the agent's reference must be updated to match. A reference to a skill that no longer exists is a `UPDATE NEEDED` item.

### Skills (`skills/*/SKILL.md`)

For each skill file, evaluate three areas:

1. **Trigger phrases** — Do the `description` field's trigger phrases still match the situations where the skill should be invoked? If this session broadened or narrowed the feature the skill covers, the triggers may be incomplete or misleading.

2. **File references** — Does the SKILL.md reference any files in `references/`? Confirm each referenced file still exists at the expected path. A broken reference means the skill's "load these for detail" instruction silently fails.

3. **Scope change** — If new functionality was added this session that falls within this skill's domain, does the skill body reflect it? A skill whose Quick Reference table omits a command added this session is incomplete.

### Commands (`commands/*.md`)

For each command file, evaluate two areas:

1. **Code path references** — Does the command reference file paths, function names, module imports, or CLI flags that changed this session? Paths that were renamed or moved, functions that were refactored out, and flags that were deprecated are `UPDATE NEEDED`.

2. **API freshness** — Does the command invoke any external APIs with request shapes that changed this session? Parameter renames, endpoint changes, and new required fields are `UPDATE NEEDED`.

### MCP Config (`.mcp.json`)

Evaluate two areas:

1. **Environment variables** — Were any environment variable names changed or removed this session? Does `.mcp.json` still reference the correct variable names? A renamed env var that the MCP config still references by the old name is a silent failure.

2. **New tool configurations** — Were any new MCP-backed tools introduced this session? If the new tools require configuration entries in `.mcp.json` that are absent, flag as `UPDATE NEEDED`.

### Manifest (`.claude-plugin/plugin.json`)

Evaluate two areas:

1. **Scope description** — Were new agents, skills, or commands created this session? Does the manifest's `description` field still accurately characterize what the plugin provides? A plugin that grew from "session closing" to "session closing + plugin sync" needs its description updated.

2. **Component registration** — If the manifest explicitly lists component files (not relying on auto-discovery), confirm all new components added this session are registered. An unregistered component will not load.

---

## Step 5: Produce the Audit Table

After auditing all components, produce a structured table before executing any changes:

```
| Component | File                                   | Verdict      | Reason                                           |
|-----------|----------------------------------------|--------------|--------------------------------------------------|
| Agent     | agents/dev-session-closer-agent.md     | CURRENT      | Description unchanged; no new tools added        |
| Skill     | skills/session-plugin-sync/SKILL.md    | CURRENT      | Created this session; content already current    |
| Command   | commands/close-session.md              | UPDATE NEEDED| References path renamed in this session          |
| MCP       | .mcp.json                              | REVIEW       | Env var renamed; new name not confirmed stable   |
| Manifest  | .claude-plugin/plugin.json             | UPDATE NEEDED| New skill added; description scope is stale      |
```

Present this table to the user. Do not execute updates silently — the table is the checkpoint before action.

**Verdicts:**
- `CURRENT` — accurately reflects current state; no changes needed
- `UPDATE NEEDED` — has confirmed staleness; apply targeted update
- `REVIEW` — change is ambiguous or requires human judgment; surface it

---

## Step 6: Execute Targeted Updates

For every `UPDATE NEEDED` item:
1. Read the current file in full
2. Identify the specific stale section (description field, tools array, file path, env var name, etc.)
3. Apply a minimal, targeted edit — update only the stale content; do not rewrite surrounding sections that remain accurate
4. Confirm the edit directly addresses the staleness reason recorded in the audit table

For every `REVIEW` item:
1. Describe the specific ambiguity in plain language
2. Present the current content alongside the proposed change
3. Wait for the user to decide: apply, skip, or modify the proposed change
4. Do not apply `REVIEW` items unilaterally

Leave `CURRENT` items untouched.

---

## Cascade Check

After updating any item, check for cascading staleness before closing:

- **Skill updated** (name change, scope change, trigger phrase change): check every agent that references the skill by name — agent descriptions and system prompt references may need updating
- **New component created this session**: confirm the manifest description covers its scope and, if components are registered explicitly, confirm it is listed
- **Agent tools list updated**: verify the agent's system prompt is internally consistent with the new tools list — a prompt that instructs use of a tool not in the list is a REVIEW item

Apply the same verdict framework to each cascading item discovered. Add rows to the audit table for each cascade item found.

---

## Common Mistakes

**Rewriting accurate content** — Only update what the session changed. If a component was accurate before the session and the session did not touch its domain, leave it alone.

**Treating REVIEW as UPDATE NEEDED** — REVIEW exists because some changes are genuinely ambiguous. Do not guess. Surface these for human decision; do not silently apply a guess.

**Missing the cascade** — Updating a skill without checking which agents reference it by name is incomplete. The cascade check is mandatory, not optional.

**Auditing without session context** — The audit is meaningless without reading the session log and git diff first. Load both sources before beginning Step 4. Without context, verdicts are guesses.

**Flagging cosmetic differences as UPDATE NEEDED** — Wording that differs from the implementation but still accurately describes it is not stale. Only flag genuine inaccuracies or missing information.

---

## References

For detailed per-component staleness signal tables and explicit signals for what NOT to update, consult:
- **`references/component-audit-rules.md`** — Staleness signals for each component type

For the full three-verdict decision framework, cascading update logic, and edge cases including brand-new components, consult:
- **`references/update-criteria.md`** — Criteria for CURRENT, UPDATE NEEDED, and REVIEW verdicts

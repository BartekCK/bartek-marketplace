---
name: ai-knowledge-sync
user-invocable: true
description: |
  Audits and updates all AI/agent documentation files to ensure consistency and accuracy after implementation work. Covers agent definitions, skills, workflows, rules files, README, and AI-specific config files across multiple AI coding tools (Claude Code, Cursor, OpenCode, Antigravity, Gemini, GitHub Copilot).

  Use this skill when: "sync AI docs", "update agent files", "check if docs are current", "update rules files", "ensure docs consistency", "align AI instructions with code", or before finishing a development session / creating a final commit.
---

# AI Knowledge Sync

## Purpose

AI/agent documentation drifts from the actual implementation after every development session. Agent descriptions reference stale tools, skills point to files that no longer exist, rules files contradict each other, and READMEs no longer reflect current capabilities. This skill audits all AI-related documentation files and applies targeted updates to restore consistency.

**Core principle:** Only update what the session actually changed. Leave everything else intact.

---

## Step 1: Detect AI Documentation Files

Scan the repository for all AI/agent-related documentation. Different AI tools use different file locations and formats.

### Discovery Checklist

| Category | Files to Find |
|----------|--------------|
| Agent definitions | `agents/*.md`, `agents/**/*.md` |
| Skills | `skills/*/SKILL.md`, `skills/**/*.md` |
| Workflow docs | `workflows/*.md`, `workflows/**/*.md` |
| Rules / instructions | `CLAUDE.md`, `.cursorrules`, `.cursor/rules/*.mdc`, `.github/copilot-instructions.md`, `GEMINI.md`, `.antigravity/**`, `.opencode/**`, `rules/*.md` |
| README | `README.md` |
| Plugin manifests | `.claude-plugin/plugin.json` |
| Commands | `commands/*.md` |
| MCP config | `.mcp.json` |

Glob for each pattern from the project root. Build a complete inventory before proceeding. Record which categories have files and which are empty. An empty category is not an error.

For tool-specific file formats and locations, consult the references:
- **`references/claude-code.md`** — Claude Code plugin structure (agents, skills, commands, plugin.json, .mcp.json)
- **`references/cursor.md`** — Cursor rules and project configuration
- **`references/opencode.md`** — OpenCode configuration files
- **`references/antigravity.md`** — Antigravity project setup
- **`references/gemini.md`** — Gemini instructions and configuration

---

## Step 2: Load Session Context

Load two sources of context before beginning any audit:

**Source 1 — Most recent session document**

Glob for `sessions/*.md` and read the highest-numbered file. If no `sessions/` directory exists, check for a legacy `session-log.md` file instead. Extract:
- Features or components implemented this session
- Files created, moved, or deleted
- API, tool, or environment variable changes
- New agents, skills, or commands added
- Any explicitly noted deprecations or removals

**Source 2 — Git diff**

Run `git diff --stat HEAD~1 HEAD` (or `git diff --stat HEAD` for unstaged changes). If `HEAD~1` fails (e.g., initial commit), fall back to `git diff --stat HEAD` only. If the session involved multiple commits, expand the range to cover all session commits. Extract:
- Which files changed
- Whether any documentation files themselves appear in the diff

Combine both into a session change summary. This summary is the ground truth used throughout the audit. If the session log and git diff contradict each other, prefer the session log — it reflects decisions made, not just file changes.

---

## Step 3: Audit Each Documentation File

Audit every file in the inventory against the session change summary. For each file, determine one of three verdicts:

- **`CURRENT`** — accurately reflects current state; no changes needed
- **`UPDATE NEEDED`** — has confirmed staleness; apply targeted update
- **`REVIEW`** — change is ambiguous or requires human judgment; surface it

### 3a. Agent Definitions

For each agent file, evaluate:

1. **Description accuracy** — Does the description accurately reflect what the agent does after this session? Check for new capabilities added or responsibilities changed.
2. **Tools list** — Does the tools array include all tools the agent now requires? Missing tools cause runtime failures.
3. **Skill/workflow references** — Does the agent reference any skills or workflows by name? If those were renamed, created, or restructured this session, references must match.

### 3b. Skills

For each skill file, evaluate:

1. **Trigger phrases** — Do the description's trigger phrases match situations where the skill should be invoked?
2. **File references** — Does the skill reference files in `references/`? Confirm each still exists at the expected path.
3. **Scope change** — If new functionality was added that falls within this skill's domain, does the skill body reflect it?

### 3c. Rules / Instruction Files

These are the AI-tool-specific files (CLAUDE.md, .cursorrules, GEMINI.md, etc.). For each:

1. **Terminology consistency** — Are the same concepts described using the same terms across all rules files? A feature called "session sync" in CLAUDE.md but "doc audit" in .cursorrules is confusing.
2. **Referenced workflows/skills exist** — Do the rules reference agents, skills, or workflows that still exist?
3. **No contradictory instructions** — Do different rules files give conflicting guidance about the same topic?
4. **Current capabilities** — Does each file reflect the project's current capabilities, not stale ones?

### 3d. Workflow Documentation

For each workflow file:

1. **Steps still valid** — Does every step reference tools, commands, and files that still exist?
2. **Order still correct** — If the process changed this session, does the workflow reflect the new order?

### 3e. Commands

For each command file:

1. **Code path references** — Does the command reference file paths, functions, or CLI flags that changed this session?
2. **API freshness** — Does the command invoke APIs with request shapes that changed?

### 3f. MCP Config

1. **Environment variables** — Were any env var names changed or removed? Does `.mcp.json` still reference correct names?
2. **New tool configurations** — Were new MCP tools introduced that need config entries?

### 3g. Plugin Manifest

1. **Scope description** — Does the manifest description cover all agents, skills, and commands the plugin provides?
2. **Component registration** — If the manifest lists components explicitly, are all new ones registered?

### 3h. README

1. **Capabilities described** — Does the README reflect current agent/AI capabilities?
2. **Setup instructions** — Are any AI-tool-specific setup steps still accurate?
3. **No TODOs left** — Check for leftover TODO markers in agent documentation sections.

---

## Step 4: Cross-File Consistency Check

After auditing individual files, check consistency across the entire documentation set:

| Check | What to Look For |
|-------|-----------------|
| Terminology | Same concept described with same terms everywhere |
| Cross-references | Links/references between files are valid and bidirectional where expected |
| Duplication | Same instructions duplicated across files — flag for consolidation |
| Contradictions | File A says "always do X", file B says "never do X" |
| Coverage gaps | A capability documented in one place but missing from related files |

---

## Step 5: Produce the Audit Table

Before executing any changes, produce a structured table:

```
| Category        | File                              | Verdict        | Reason                                          |
|-----------------|-----------------------------------|----------------|------------------------------------------------|
| Agent           | agents/my-agent.md                | CURRENT        | Description unchanged; no new tools added       |
| Skill           | skills/my-skill/SKILL.md          | UPDATE NEEDED  | References renamed file path                    |
| Rules           | CLAUDE.md                         | UPDATE NEEDED  | Missing new architectural decision              |
| Rules           | .cursorrules                      | REVIEW         | May need new pattern added                      |
| README          | README.md                         | UPDATE NEEDED  | Capabilities section outdated                   |
| Manifest        | .claude-plugin/plugin.json        | CURRENT        | Auto-discovery; no explicit list to update      |
```

Present this table to the user. Do not execute updates silently — the table is the checkpoint before action.

---

## Step 6: Execute Targeted Updates

For every `UPDATE NEEDED` item:
1. Read the current file in full
2. Identify the specific stale section
3. Apply a minimal, targeted edit — update only the stale content
4. Confirm the edit addresses the staleness reason from the audit table

For every `REVIEW` item:
1. Describe the ambiguity in plain language
2. Present the current content alongside the proposed change
3. Wait for the user to decide: apply, skip, or modify

Leave `CURRENT` items untouched.

---

## Step 7: Cascade Check

After updating any item, check for cascading staleness:

- **Skill updated** (name/scope/trigger change): check every agent and rules file that references the skill
- **New component created**: confirm manifests and README cover its scope
- **Agent tools list updated**: verify the agent's prompt is consistent with the new tools list
- **Rules file updated**: check other rules files for contradictions with the new content
- **Env var renamed**: check MCP config and command files for stale references

Apply the same verdict framework to each cascading item. Add rows to the audit table.

---

## Step 8: Final Validation

Before declaring sync complete, verify:

- [ ] No TODOs left in agent documentation files
- [ ] No contradictory instructions across rules files
- [ ] README reflects current agent/AI capabilities
- [ ] All cross-references between files resolve to existing targets
- [ ] Terminology is consistent across all documentation files

---

## Documentation Hygiene Rules

- Keep heading structure consistent within each file category
- Remove outdated instructions rather than commenting them out
- Ensure code examples in documentation are syntactically correct
- When removing a section, check if other files reference it first

---

## Common Mistakes

**Rewriting accurate content** — Only update what the session changed. If a file was accurate before and the session didn't touch its domain, leave it alone.

**Treating REVIEW as UPDATE NEEDED** — REVIEW exists because some changes are genuinely ambiguous. Surface these for human decision.

**Missing the cascade** — Updating a skill without checking which agents reference it by name is incomplete.

**Auditing without session context** — Load both sources (session log + git diff) before beginning the audit.

**Flagging cosmetic differences** — Wording that differs from implementation but still accurately describes it is not stale.

**Forcing consistency where none is needed** — Different AI tools have different conventions. A `.cursorrules` file doesn't need to match CLAUDE.md's structure — only the facts and terminology should be consistent.

**Trusting reference files blindly** — The reference files in `references/` describe expected file formats and locations for each AI tool. If a discovered file's actual format does not match what the reference describes, treat the reference as potentially stale and audit based on the file's actual format. Reference files should be updated when discrepancies are found.

---

## Verdict Decision Framework

When deciding between verdicts, apply this test:

> Can you identify the specific session change (from the session log or git diff) that caused this inaccuracy?

- **Yes, specific cause identified** -> `UPDATE NEEDED`
- **Inaccuracy present but cause unclear** -> `REVIEW`
- **No inaccuracy, only speculation about future staleness** -> `CURRENT`

---

## References

For AI-tool-specific file formats, locations, and conventions:
- **`references/claude-code.md`** — Claude Code plugins, agents, skills, commands, .mcp.json
- **`references/cursor.md`** — Cursor rules (.cursorrules, .cursor/rules/*.mdc)
- **`references/copilot.md`** — GitHub Copilot instructions
- **`references/opencode.md`** — OpenCode configuration
- **`references/antigravity.md`** — Antigravity project setup
- **`references/gemini.md`** — Gemini CLI and Gemini Code Assist

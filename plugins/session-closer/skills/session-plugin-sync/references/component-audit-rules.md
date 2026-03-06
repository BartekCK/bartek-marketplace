# Component Audit Rules

Per-component staleness signal tables. Use these when applying Step 4 of the session-plugin-sync skill. Each table lists signals that indicate a component needs updating, paired with the verdict to assign.

---

## Agents (`agents/*.md`)

### Staleness Signals

| Signal | Verdict |
|--------|---------|
| Description omits a capability added this session | UPDATE NEEDED |
| Description describes a capability removed or restructured this session | UPDATE NEEDED |
| `tools:` list is missing a tool the agent now uses | UPDATE NEEDED |
| `tools:` list contains a tool the agent no longer uses | REVIEW |
| System prompt references a skill by a name that changed this session | UPDATE NEEDED |
| System prompt references a skill that was removed this session | UPDATE NEEDED |
| System prompt references a skill created this session (by correct name) | CURRENT |
| Agent was directly modified in the git diff this session | REVIEW (verify intent) |
| New skill created this session that this agent should invoke | REVIEW |

### What NOT to Update in Agents

- Wording that differs stylistically from implementation but remains accurate
- `color:` and `model:` fields — these are not affected by implementation changes
- Example blocks in the description — only update if the examples describe workflows that no longer exist
- Historical trigger phrases that still apply, even if new ones could be added

---

## Skills (`skills/*/SKILL.md`)

### Staleness Signals

| Signal | Verdict |
|--------|---------|
| Description trigger phrases omit a new use case added this session | UPDATE NEEDED |
| Description trigger phrases describe a use case removed this session | UPDATE NEEDED |
| References section points to a file that was renamed or deleted | UPDATE NEEDED |
| Quick Reference table omits a command, flag, or pattern added this session | UPDATE NEEDED |
| Quick Reference table includes a command or flag removed this session | UPDATE NEEDED |
| Body describes a step or pattern that was refactored out this session | UPDATE NEEDED |
| Skill was directly modified in the git diff this session | CURRENT (already updated) |
| Skill was created this session | CURRENT |
| Scope technically broadened but skill still accurate for its documented scope | REVIEW |

### What NOT to Update in Skills

- Prose explanations that remain conceptually accurate even if wording could be tightened
- Examples that illustrate a pattern still valid, even if not the only valid approach
- Frontmatter `name:` field — renaming a skill is a breaking change; treat as REVIEW, not automatic update
- Reference file contents unless explicitly changed by session work

---

## Commands (`commands/*.md`)

### Staleness Signals

| Signal | Verdict |
|--------|---------|
| Command references a file path that was moved or renamed this session | UPDATE NEEDED |
| Command references a function or symbol removed or renamed this session | UPDATE NEEDED |
| Command invokes a CLI tool with flags that changed this session | UPDATE NEEDED |
| Command invokes an API endpoint whose request shape changed this session | UPDATE NEEDED |
| Command references a module import path that changed this session | UPDATE NEEDED |
| Command output format description no longer matches actual output | UPDATE NEEDED |
| Command references a tool or binary that was removed from the project | UPDATE NEEDED |
| Command description omits a new argument or option added this session | UPDATE NEEDED |
| Command behavior unchanged; only implementation internals changed | CURRENT |

### What NOT to Update in Commands

- Descriptions that accurately describe the command's effect even if the underlying implementation changed
- Usage examples that remain valid even if the internals differ
- Help text for stable flags that weren't touched this session

---

## MCP Config (`.mcp.json`)

### Staleness Signals

| Signal | Verdict |
|--------|---------|
| References an env var by a name that was renamed this session | UPDATE NEEDED |
| References an env var that was removed this session | UPDATE NEEDED |
| Missing a configuration entry for a new MCP-backed tool added this session | UPDATE NEEDED |
| Server `command` or `args` reference a path that changed this session | UPDATE NEEDED |
| New MCP server type was adopted but config block is absent | UPDATE NEEDED |
| Env var was renamed but the new name is not yet confirmed stable | REVIEW |
| Config references a server not used this session, status unknown | REVIEW |
| Config is unchanged and no env vars or tools were modified | CURRENT |

### What NOT to Update in MCP Config

- Server entries for tools not touched this session, even if their env vars look unusual
- Optional configuration fields that default correctly — do not add defaults that aren't needed
- Comments documenting server purpose — only update if the server's role changed

---

## Manifest (`.claude-plugin/plugin.json`)

### Staleness Signals

| Signal | Verdict |
|--------|---------|
| Description omits a capability introduced by a new agent or skill this session | UPDATE NEEDED |
| Description describes a capability whose agent or skill was removed this session | UPDATE NEEDED |
| New agent, skill, or command was created but is not registered (if explicit list used) | UPDATE NEEDED |
| Plugin version not incremented after a feature addition (if project uses versioning) | REVIEW |
| New agent created but the manifest description still accurately covers the plugin's scope | CURRENT |
| No new components added or removed this session | CURRENT |

### What NOT to Update in Manifest

- Technical metadata fields (`id`, `author`, `license`) — these are not affected by implementation changes
- `keywords` or `tags` unless they actively misdescribe the plugin after the session
- Auto-discovery configuration — do not switch from auto-discovery to explicit lists without user direction
- The manifest version scheme — do not change versioning strategy during routine session sync

---

## Cross-Component Rules

These signals span multiple component types:

| Situation | Affected Components | Verdict |
|-----------|---------------------|---------|
| Skill renamed this session | All agents referencing the old name | UPDATE NEEDED |
| Agent renamed this session | Manifest (if agents listed explicitly) | UPDATE NEEDED |
| New skill created this session | Agents that should invoke it, manifest description | REVIEW |
| Core abstraction refactored this session | Commands referencing old paths, skill Quick Reference | UPDATE NEEDED |
| Environment variable renamed | `.mcp.json`, any command scripts referencing it | UPDATE NEEDED |

# Update Criteria

Decision framework for the three-verdict system used in session-plugin-sync. Apply these criteria when a staleness signal is ambiguous or when the component-audit-rules tables leave room for interpretation.

---

## The Three Verdicts

### CURRENT

Assign `CURRENT` when a component accurately describes the state of the codebase after this session and contains no missing information relevant to the session's changes.

**Criteria for CURRENT:**
- Every capability the component describes still exists and works as described
- No new capabilities introduced by this session are absent from the component
- All file paths, symbol names, env vars, and tool references are valid after the session
- Skill references in agents point to skill names that still exist

**CURRENT does not require perfection.** A component can have minor wording differences from the implementation and still be CURRENT — accuracy matters, not stylistic match. A description that says "analyzes code structure" for an agent that does slightly more than that is still CURRENT if the core characterization is right.

---

### UPDATE NEEDED

Assign `UPDATE NEEDED` when there is a **confirmed inaccuracy or confirmed omission** directly caused by this session's changes.

**Criteria for UPDATE NEEDED:**

1. **Omits new capability** — The session added a feature, tool, command, or behavior that the component should document but doesn't. The omission would cause a future Claude instance to misuse or underuse the component.

2. **References stale state** — The component names a file, path, symbol, env var, or skill that no longer exists or was renamed this session. The reference is now broken or misleading.

3. **New component exists that should be referenced** — A new agent, skill, or command was created this session. Another component that orchestrates or invokes it (e.g., an agent that should reference the new skill) does not reference it yet.

4. **Tools list gap** — An agent's `tools:` list is missing a tool the agent now demonstrably needs based on what was implemented this session.

**UPDATE NEEDED requires confidence.** You must be able to point to a specific session change (from the session log or git diff) that caused the inaccuracy. If you cannot identify the specific cause, use REVIEW instead.

---

### REVIEW

Assign `REVIEW` when a change is real but the correct update is ambiguous, the change may not be stable yet, or the decision has implications beyond a routine description update.

**Criteria for REVIEW:**

1. **Tangential change** — The session changed something in the component's vicinity, but the component itself may still be accurate. The connection is indirect. Example: a new skill was created adjacent to an existing skill's domain, but the existing skill's scope is not clearly wrong.

2. **Ambiguous stability** — A renamed identifier (env var, function, path) appears in the git diff, but the rename was part of an in-progress refactor that may not be final. Updating references to an unstable name may require re-updating next session.

3. **Conflicting signals** — The session log describes one set of changes but the git diff shows a different set. The two sources disagree on what actually changed.

4. **Scope expansion that requires judgment** — A new capability was added that the component could mention, but adding it would meaningfully change the component's scope. This warrants a human decision, not an automatic update.

5. **Tool removal from agent** — Removing a tool from an agent's `tools:` list is a potentially breaking change. Always surface for REVIEW rather than applying automatically.

6. **Skill renaming** — Renaming a skill cascades to all referencing agents. Surface as REVIEW with full cascade impact listed.

---

## Cascading Update Logic

Some updates trigger cascading checks. Apply these rules after completing the initial audit:

**When a skill is updated (name change, scope change, trigger phrase change):**
Check every agent that references the skill by name in its system prompt. Each reference is a potential cascade item. Assign CURRENT if the reference uses the new name, UPDATE NEEDED if it uses the old name, REVIEW if the scope change is ambiguous.

**When a new component is created this session:**
Check the manifest description. If the manifest describes the plugin's scope and the new component expands that scope, the manifest is a cascade UPDATE NEEDED item. If the new component is within the existing scope, the manifest is CURRENT.

**When an agent's tools list is updated:**
Verify the agent's system prompt does not instruct use of a tool absent from the updated list. An instruction to use a tool not in the list is a REVIEW item (the system prompt may need to be updated or the tools list may need the tool added back).

**When an env var is renamed in `.mcp.json`:**
Check every command file that references the variable by name. Each stale reference is UPDATE NEEDED.

---

## Edge Case: Brand-New Components

Components created during this session require special handling:

- **Newly created components are always CURRENT for their own file** — the file was just written; it cannot be stale with respect to itself.
- **Newly created components may trigger UPDATE NEEDED in other components** — the manifest, referencing agents, and orchestrating commands may need to acknowledge them.
- **Do not audit newly created component files for internal accuracy** — that content was written this session; treat it as intentional. Only audit pre-existing files for staleness caused by the session.

---

## The Confidence Threshold

When deciding between `UPDATE NEEDED` and `REVIEW`, apply this test:

> Can you identify the specific session change (name it from the session log or git diff) that caused this inaccuracy?

- **Yes, specific cause identified** → `UPDATE NEEDED`
- **Inaccuracy present but cause unclear** → `REVIEW`
- **No inaccuracy, only speculation about future staleness** → `CURRENT`

Do not assign `UPDATE NEEDED` based on speculation. If the inaccuracy exists but you cannot trace it to a specific session change, surface it as `REVIEW` so the user can decide with full context.

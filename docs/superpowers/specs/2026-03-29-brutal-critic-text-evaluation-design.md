# Brutal Critic: Text & Thought Evaluation Expansion

**Date:** 2026-03-29
**Status:** Approved

## Summary

Expand the `brutal-critic` plugin from code-only evaluation to a universal quality evaluator covering code, technical writing, general writing, communication, and thought/idea evaluation. The existing agent is renamed and a new text-focused agent is added. Five domain-specific evaluation skills provide the scoring frameworks.

## Architecture

### Plugin: `brutal-critic` (existing, updated)

Single plugin, two agents, five skills. The plugin description changes to reflect broader scope.

### Agents

#### `brutal-critic-code-agent` (renamed from `brutal-critic-agent`)

- **Purpose:** Evaluate code quality
- **Tools:** Read, Glob, Grep, WebSearch, WebFetch, Skill
- **Model:** opus
- **Color:** red
- **Process:** Read code -> research best practices online -> score per category -> PASS/FAIL verdict with fixes
- **Skill:** `brutal-evaluation-code`
- **Changes from current:** Renamed, references updated skill name, linting category added

#### `brutal-critic-text-agent` (new)

- **Purpose:** Evaluate any non-code content with auto-detection
- **Tools:** Read, Glob, WebSearch, WebFetch, Skill
- **Model:** opus
- **Color:** red
- **Process:** Read/receive text -> auto-detect type -> load matching skill -> score per category -> PASS/FAIL verdict + improved rewrite of weak sections
- **Auto-detection targets:**
  - Technical writing (RFCs, ADRs, READMEs, architecture docs, diagrams) -> `brutal-evaluation-technical-writing`
  - General writing (articles, blog posts, general text) -> `brutal-evaluation-writing`
  - Communication (emails, Slack, proposals, reports) -> `brutal-evaluation-communication`
  - Thoughts/ideas (arguments, reasoning, strategies) -> `brutal-evaluation-thought`

### Skills (all agent-exclusive: `user-invocable: false` + `disable-model-invocation: true`)

#### 1. `brutal-evaluation-code` (renamed from `brutal-evaluation`)

9 categories:
1. Correctness
2. Error Handling
3. Architecture & Design
4. Readability & Maintainability
5. Security
6. Performance
7. Testing
8. Linting & Formatting (NEW)
9. API Design (when applicable)

#### 2. `brutal-evaluation-technical-writing` (new)

7 categories:
1. Accuracy & Correctness
2. Structure & Organization
3. Clarity & Conciseness
4. Completeness
5. Audience Awareness
6. Examples & Visual Aids
7. Actionability

#### 3. `brutal-evaluation-writing` (new)

6 categories:
1. Grammar & Mechanics
2. Clarity & Conciseness
3. Structure & Flow
4. Tone & Voice
5. Argumentation & Logic
6. Engagement & Readability

#### 4. `brutal-evaluation-communication` (new)

6 categories:
1. Clarity of Purpose
2. Tone & Professionalism
3. Brevity
4. Call to Action
5. Audience Awareness
6. Formatting & Scannability

#### 5. `brutal-evaluation-thought` (new)

6 categories:
1. Logical Coherence
2. Evidence & Support
3. Completeness
4. Feasibility
5. Originality
6. Counter-arguments Considered

### Common elements across all skills

- Rating scale: 1-10 with defined meanings
- Minimum PASS threshold: 8/10
- Weakest-link overall scoring (overall = lowest category)
- CRITICAL issue caps category at 5/10
- Mandatory "What Was Done Right" section
- Report template with scores table, issues (CRITICAL/MAJOR/MINOR), references, verdict

### Text-skill additions (not in code skill)

- **"Improved Version" section** — rewrites weak parts showing how they should read
- **Advisory suggestions** — specific improvement hints beyond scoring

## File structure

```
plugins/brutal-critic/
  .claude-plugin/plugin.json                          # updated description
  README.md                                           # updated
  agents/
    brutal-critic-code-agent.md                       # renamed from brutal-critic-agent.md
    brutal-critic-text-agent.md                       # new
  skills/
    brutal-evaluation-code/SKILL.md                   # renamed from brutal-evaluation/
    brutal-evaluation-technical-writing/SKILL.md      # new
    brutal-evaluation-writing/SKILL.md                # new
    brutal-evaluation-communication/SKILL.md          # new
    brutal-evaluation-thought/SKILL.md                # new
```

## Migration

- `git mv` the existing agent and skill files to preserve history
- Update marketplace.json description
- Update plugin.json description
- Delete old directories after move

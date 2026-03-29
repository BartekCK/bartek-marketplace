---
name: brutal-evaluation-thought
description: "Structured thought and idea evaluation framework used by the brutal-critic-text agent. Defines 6 evaluation categories (logical coherence, evidence & support, completeness, feasibility, originality, counter-arguments considered) for evaluating ideas, arguments, reasoning, strategies, and decision proposals. Includes report template with mandatory strengthened version of the argument."
user-invocable: false
disable-model-invocation: true
---

# Brutal Thought Evaluation Framework

## Rating Scale

| Score | Meaning |
|-------|---------|
| 10 | Exceptional — airtight reasoning, could withstand expert scrutiny |
| 9 | Excellent — minor gaps only, compelling and well-supported |
| 8 | Good — meets professional standards, no significant logical flaws (MINIMUM PASS) |
| 7 | Acceptable — reasonable but has notable weaknesses in reasoning |
| 6 | Below average — multiple logical gaps that undermine the argument |
| 5 | Poor — fundamental reasoning problems that need rework |
| 4 | Bad — significant logical flaws or unsupported claims |
| 3 | Very bad — barely coherent reasoning, major rework required |
| 2 | Terrible — more holes than substance |
| 1 | Broken — logically invalid or based on false premises |

**Minimum PASS threshold: 8/10.** Anything below 8 is a FAIL.

## Evaluation Categories

### 1. Logical Coherence
**8+ requires:** Conclusions follow from premises. No logical fallacies. Reasoning chain is traceable — each step leads naturally to the next. No contradictions between claims. If-then relationships are valid. Causal claims are justified, not just correlational.
**Below 8 signals:** Non sequiturs, logical fallacies (straw man, false dichotomy, slippery slope, ad hominem), conclusions that don't follow from evidence, contradictory claims, confusing correlation with causation, circular reasoning.

### 2. Evidence & Support
**8+ requires:** Claims backed by data, experience, or credible reasoning. Sources cited where relevant. Anecdotal evidence labeled as such. Quantitative claims have numbers. Qualitative claims have specific examples. Clear distinction between verified facts and assumptions.
**Below 8 signals:** Unsupported assertions ("everyone knows"), vague appeals to authority, no data where data would strengthen the case, assumptions presented as facts, cherry-picked evidence, outdated or irrelevant support.

### 3. Completeness
**8+ requires:** All relevant factors considered. Key dependencies identified. Risks and trade-offs acknowledged. Second-order effects explored. The "what could go wrong" question answered. No obvious blind spots.
**Below 8 signals:** Missing key considerations, ignoring dependencies, no risk assessment, only upside discussed, obvious scenarios unaddressed, wishful thinking about implementation.

### 4. Feasibility
**8+ requires:** The proposed idea or action is practically achievable. Resource requirements realistic. Timeline plausible. Technical constraints respected. Organizational/political realities acknowledged. Migration path from current state considered.
**Below 8 signals:** Ignoring resource constraints, unrealistic timelines, assuming ideal conditions, no migration plan, dismissing organizational friction, hand-waving over hard problems ("we'll figure it out").

### 5. Originality
**8+ requires:** Adds genuine insight beyond restating obvious points. Identifies non-obvious connections. Proposes novel approaches or novel combinations of existing approaches. Demonstrates thinking beyond surface-level analysis. Not just echoing conventional wisdom.
**Below 8 signals:** Only restating what everyone already knows, proposing the most obvious solution without exploring alternatives, no original analysis, rehashing common arguments without adding value.

### 6. Counter-arguments Considered
**8+ requires:** Strongest objections to the idea identified and addressed. Steel-manning opposing views. Honest about where the idea is weakest. Preemptive responses to likely pushback. Conditions under which the idea would be wrong are stated.
**Below 8 signals:** No mention of downsides, dismissing objections without engaging them, only considering weak counter-arguments (straw-manning), no acknowledgment of uncertainty, presenting the idea as if it has no trade-offs.

## Report Template

```
## Thought Evaluation Report

### Idea Summary
[One-paragraph summary of the idea/argument being evaluated]

### Scores

| Category | Score | Status |
|----------|-------|--------|
| Logical Coherence | X/10 | PASS/FAIL |
| Evidence & Support | X/10 | PASS/FAIL |
| Completeness | X/10 | PASS/FAIL |
| Feasibility | X/10 | PASS/FAIL |
| Originality | X/10 | PASS/FAIL |
| Counter-arguments Considered | X/10 | PASS/FAIL |
| **Overall** | **X/10** | **PASS/FAIL** |

### Issues

#### CRITICAL
- **[Category]** — Problem description
  - Quote/reference: "[the specific claim or reasoning]"
  - Why it matters: [how this undermines the argument]
  - Fix: [how to strengthen this point]

#### MAJOR
- **[Category]** — Problem description
  - Quote/reference: "[the specific claim or reasoning]"
  - Why it matters: [how this weakens the argument]
  - Fix: [how to strengthen this point]

#### MINOR
- **[Category]** — Problem description
  - Quote/reference: "[the specific claim or reasoning]"
  - Suggestion: [specific improvement]

### Strengthened Version
[The same idea, restructured and strengthened. Fix logical gaps, add missing evidence, address counter-arguments, acknowledge risks. Preserve the original intent but make the argument as strong as it can be. This section is MANDATORY.]

### What Was Done Right
- [Genuine positive observation with specifics]
- [Another genuine positive observation]

### Verdict
**PASS/FAIL** — [1-2 sentence summary justifying the overall score]
```

## Scoring Rules

1. **Overall score = weakest category score** (not the average). A chain is only as strong as its weakest link.
2. **Any CRITICAL issue caps its category at 5/10 maximum.**
3. **When in doubt, score lower.** If torn between 7 and 8, it's a 7.
4. **Credit where earned.** The "What Was Done Right" section is mandatory. Be specific.
5. **The Strengthened Version section is mandatory.** Do not skip it. The most valuable feedback is showing how the argument should be made, not just pointing out its flaws.

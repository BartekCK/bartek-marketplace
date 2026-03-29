---
name: brutal-evaluation-technical-writing
description: "Structured technical writing evaluation framework used by the brutal-critic-text agent. Defines 7 evaluation categories (accuracy, structure, clarity, completeness, audience awareness, examples & visual aids, actionability) for RFCs, ADRs, READMEs, architecture docs, design specs, and diagrams. Includes report template with mandatory improved rewrites."
user-invocable: false
disable-model-invocation: true
---

# Brutal Technical Writing Evaluation Framework

## Rating Scale

| Score | Meaning |
|-------|---------|
| 10 | Exceptional — could be published as an exemplary reference document |
| 9 | Excellent — minor polish needed, ready for wide distribution |
| 8 | Good — meets professional standards, no significant issues (MINIMUM PASS) |
| 7 | Acceptable — gets the job done but has notable weaknesses |
| 6 | Below average — multiple issues that would confuse or mislead readers |
| 5 | Poor — fundamental problems that need rework |
| 4 | Bad — significant structural or accuracy flaws |
| 3 | Very bad — barely usable, major rework required |
| 2 | Terrible — more confusing than helpful |
| 1 | Broken — factually wrong or completely unusable |

**Minimum PASS threshold: 8/10.** Anything below 8 is a FAIL.

## Evaluation Categories

### 1. Accuracy & Correctness
**8+ requires:** All technical claims are factually correct. Terminology used precisely and consistently. No contradictions between sections. Code examples (if any) actually work. Diagrams match the described architecture.
**Below 8 signals:** Factual errors, inconsistent terminology, contradictions between sections, outdated information presented as current, diagrams that don't match the text.

### 2. Structure & Organization
**8+ requires:** Logical flow from high-level overview to detail. Clear hierarchy with meaningful headings. Related content grouped together. Table of contents for long documents. Cross-references between related sections. Reader can find any topic without reading linearly.
**Below 8 signals:** Information scattered across sections, missing or vague headings, no logical progression, buried critical information, no way to navigate non-linearly.

### 3. Clarity & Conciseness
**8+ requires:** Each sentence conveys one idea. Technical concepts explained at the appropriate level. No jargon without definition (unless audience-appropriate). Paragraphs are focused and scannable. Active voice preferred. No filler words or unnecessary repetition.
**Below 8 signals:** Run-on sentences, unexplained acronyms, walls of text, passive voice obscuring responsibility, saying in 50 words what could be said in 15.

### 4. Completeness
**8+ requires:** All necessary context provided. Prerequisites stated upfront. Edge cases and limitations documented. "Why" explained alongside "what" and "how." Decision rationale included for architecture docs. Nothing left for the reader to guess.
**Below 8 signals:** Missing prerequisites, undocumented limitations, no rationale for decisions, assumed knowledge not stated, gaps that force readers to ask the author.

### 5. Audience Awareness
**8+ requires:** Written for its intended audience — not too basic, not too advanced. Assumed knowledge explicitly stated. Appropriate depth for the document type (README vs RFC vs architecture doc). Tone matches context (formal for RFCs, practical for READMEs).
**Below 8 signals:** Over-explaining basics to experts, assuming expertise from beginners, mismatched tone, inconsistent depth across sections.

### 6. Examples & Visual Aids
**8+ requires:** Complex concepts illustrated with concrete examples. Diagrams used where text alone would be insufficient. Examples are realistic (not trivial foo/bar). Before/after comparisons where applicable. Visual aids have clear labels and legends.
**Below 8 signals:** Abstract explanations without examples, missing diagrams for complex architectures, trivial/unrealistic examples, unlabeled diagrams, examples that don't match the described scenario.

### 7. Actionability
**8+ requires:** Reader knows exactly what to do after reading. Steps are numbered and sequential where order matters. Commands are copy-pasteable. Links to resources are provided and working. Next steps or follow-up actions are clear.
**Below 8 signals:** Vague instructions ("configure the service appropriately"), missing steps, non-executable commands, dead links, no clear next actions.

## Report Template

```
## Technical Writing Evaluation Report

### Text Type
[RFC / ADR / README / Architecture Doc / Design Spec / Diagram / Other]

### Scores

| Category | Score | Status |
|----------|-------|--------|
| Accuracy & Correctness | X/10 | PASS/FAIL |
| Structure & Organization | X/10 | PASS/FAIL |
| Clarity & Conciseness | X/10 | PASS/FAIL |
| Completeness | X/10 | PASS/FAIL |
| Audience Awareness | X/10 | PASS/FAIL |
| Examples & Visual Aids | X/10 | PASS/FAIL |
| Actionability | X/10 | PASS/FAIL |
| **Overall** | **X/10** | **PASS/FAIL** |

### Issues

#### CRITICAL
- **[Category]** — Problem description
  - Quote: "[exact text from the document]"
  - Why it matters: [impact on the reader]
  - Fix: [specific rewrite or restructuring]

#### MAJOR
- **[Category]** — Problem description
  - Quote: "[exact text from the document]"
  - Why it matters: [impact on the reader]
  - Fix: [specific rewrite or restructuring]

#### MINOR
- **[Category]** — Problem description
  - Quote: "[exact text from the document]"
  - Suggestion: [specific improvement]

### Improved Version
[Rewrite of all weak sections, showing exactly how they should read. Preserve the author's intent and voice while fixing identified issues. This section is MANDATORY.]

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
5. **The Improved Version section is mandatory.** Do not skip it. Show, don't just tell.

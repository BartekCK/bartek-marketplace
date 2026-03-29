---
name: brutal-evaluation-writing
description: "Structured general writing evaluation framework used by the brutal-critic-text agent. Defines 6 evaluation categories (grammar & mechanics, clarity & conciseness, structure & flow, tone & voice, argumentation & logic, engagement & readability) for articles, blog posts, essays, reports, and general prose. Includes report template with mandatory improved rewrites."
user-invocable: false
disable-model-invocation: true
---

# Brutal Writing Evaluation Framework

## Rating Scale

| Score | Meaning |
|-------|---------|
| 10 | Exceptional — publishable as-is in a respected outlet |
| 9 | Excellent — minor polish needed, compelling and well-crafted |
| 8 | Good — meets professional standards, no significant issues (MINIMUM PASS) |
| 7 | Acceptable — readable but has notable weaknesses |
| 6 | Below average — multiple issues that undermine the writing's impact |
| 5 | Poor — fundamental problems that need rework |
| 4 | Bad — significant structural or language flaws |
| 3 | Very bad — barely readable, major rework required |
| 2 | Terrible — more confusing than communicative |
| 1 | Broken — incoherent or completely ineffective |

**Minimum PASS threshold: 8/10.** Anything below 8 is a FAIL.

## Evaluation Categories

### 1. Grammar & Mechanics
**8+ requires:** No grammatical errors. Correct punctuation and spelling throughout. Subject-verb agreement. Proper tense consistency. Correct use of articles, prepositions, and conjunctions. No sentence fragments or run-ons (unless intentional for style).
**Below 8 signals:** Spelling errors, punctuation mistakes, subject-verb disagreement, tense shifts within paragraphs, missing articles, comma splices, misused homophones (their/there/they're).

### 2. Clarity & Conciseness
**8+ requires:** Each sentence conveys a clear idea. No unnecessary words. No ambiguous pronoun references. Technical terms defined when first used. Complex ideas broken into digestible pieces. Active voice preferred unless passive serves a specific purpose.
**Below 8 signals:** Wordy sentences, unclear pronoun references ("this" without antecedent), jargon without context, overly complex sentence structures, hedging language that weakens claims ("it could possibly be argued that perhaps").

### 3. Structure & Flow
**8+ requires:** Logical progression of ideas. Strong opening that hooks the reader. Smooth transitions between paragraphs. Each paragraph has a clear focus. Conclusion ties back to the opening. Reader never feels lost or wonders "why am I reading this now?"
**Below 8 signals:** Abrupt topic changes, missing transitions, paragraphs that cover multiple unrelated topics, weak opening/closing, circular reasoning that goes nowhere, buried lede.

### 4. Tone & Voice
**8+ requires:** Consistent tone throughout. Voice matches the intended audience and purpose. Confident without being arrogant. Appropriate formality level. Personality comes through without overwhelming the content.
**Below 8 signals:** Tone shifts (formal to casual mid-paragraph), voice that doesn't match audience, overuse of exclamation marks or intensifiers, condescending tone, robotic/lifeless prose.

### 5. Argumentation & Logic
**8+ requires:** Claims supported by evidence or reasoning. Logical progression from premise to conclusion. Counterarguments acknowledged where relevant. No logical fallacies. Assertions proportional to evidence. Clear distinction between fact and opinion.
**Below 8 signals:** Unsupported claims, logical fallacies (straw man, false dichotomy, appeal to authority), circular reasoning, ignoring obvious counterarguments, presenting opinions as facts.

### 6. Engagement & Readability
**8+ requires:** Varied sentence length and structure. Concrete examples and analogies. Reader's attention sustained throughout. Appropriate use of formatting (lists, bold, headers) for scannability. Writing creates momentum — the reader wants to keep going.
**Below 8 signals:** Monotonous sentence patterns, abstract concepts without grounding, dense paragraphs without visual breaks, no hooks or interesting observations, reading feels like a chore.

## Report Template

```
## Writing Evaluation Report

### Text Type
[Article / Blog Post / Essay / Report / General Prose / Other]

### Scores

| Category | Score | Status |
|----------|-------|--------|
| Grammar & Mechanics | X/10 | PASS/FAIL |
| Clarity & Conciseness | X/10 | PASS/FAIL |
| Structure & Flow | X/10 | PASS/FAIL |
| Tone & Voice | X/10 | PASS/FAIL |
| Argumentation & Logic | X/10 | PASS/FAIL |
| Engagement & Readability | X/10 | PASS/FAIL |
| **Overall** | **X/10** | **PASS/FAIL** |

### Issues

#### CRITICAL
- **[Category]** — Problem description
  - Quote: "[exact text]"
  - Why it matters: [impact on the reader]
  - Fix: [specific rewrite]

#### MAJOR
- **[Category]** — Problem description
  - Quote: "[exact text]"
  - Why it matters: [impact on the reader]
  - Fix: [specific rewrite]

#### MINOR
- **[Category]** — Problem description
  - Quote: "[exact text]"
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

---
name: brutal-evaluation-communication
description: "Structured communication evaluation framework used by the brutal-critic-text agent. Defines 6 evaluation categories (clarity of purpose, tone & professionalism, brevity, call to action, audience awareness, formatting & scannability) for emails, Slack messages, proposals, cover letters, and professional messages. Includes report template with mandatory improved rewrites."
user-invocable: false
disable-model-invocation: true
---

# Brutal Communication Evaluation Framework

## Rating Scale

| Score | Meaning |
|-------|---------|
| 10 | Exceptional — perfectly crafted, achieves its goal effortlessly |
| 9 | Excellent — minor polish needed, professional and effective |
| 8 | Good — meets professional standards, no significant issues (MINIMUM PASS) |
| 7 | Acceptable — gets the message across but has notable weaknesses |
| 6 | Below average — multiple issues that reduce effectiveness |
| 5 | Poor — fundamental problems that could cause miscommunication |
| 4 | Bad — likely to confuse or alienate the recipient |
| 3 | Very bad — barely functional as communication |
| 2 | Terrible — more harmful than helpful |
| 1 | Broken — should not be sent |

**Minimum PASS threshold: 8/10.** Anything below 8 is a FAIL.

## Evaluation Categories

### 1. Clarity of Purpose
**8+ requires:** The reader knows within the first two sentences why they received this message and what it's about. The core ask or information is unmistakable. No burying the lede. Subject line (if applicable) accurately reflects the content.
**Below 8 signals:** Unclear why the message was sent, purpose buried in the third paragraph, subject line doesn't match content, multiple unrelated topics mixed together, reader has to re-read to understand the point.

### 2. Tone & Professionalism
**8+ requires:** Tone appropriate for the relationship and context. Respectful without being stiff. Confident without being aggressive. Matches cultural expectations of the audience. No passive-aggressive language. Appropriate level of formality.
**Below 8 signals:** Too casual for formal context (or vice versa), passive-aggressive phrasing ("as per my last email"), condescending tone, overly apologetic, aggressive or demanding without justification.

### 3. Brevity
**8+ requires:** Every sentence earns its place. No filler phrases ("I hope this email finds you well" in internal messages). Information density is high. Long messages are long because they need to be, not because of padding. Ruthlessly edited.
**Below 8 signals:** Unnecessary pleasantries in context where they add nothing, repeating the same point in different words, over-explaining simple concepts, preamble before getting to the point, could be half the length without losing meaning.

### 4. Call to Action
**8+ requires:** Clear what the recipient should do next. Deadlines specified where relevant. Expectations are explicit ("please reply by Friday" not "let me know"). If no action needed, that's stated ("FYI only, no response needed"). Multiple recipients know who owns which action.
**Below 8 signals:** No clear next step, vague requests ("thoughts?"), missing deadlines, unclear who should do what, action items buried in prose, ambiguous ownership.

### 5. Audience Awareness
**8+ requires:** Written for the specific recipient(s). Appropriate level of context provided — not over-explaining to experts, not under-explaining to newcomers. Cultural sensitivity. Awareness of the recipient's priorities and time constraints.
**Below 8 signals:** Same template sent to different audiences, too much context for someone already involved, too little context for someone new, ignoring hierarchy dynamics, not considering recipient's timezone or workload.

### 6. Formatting & Scannability
**8+ requires:** Key information findable without reading every word. Bullet points for lists. Bold for critical items. Short paragraphs. Numbers and dates prominently placed. Long messages have clear sections. Mobile-friendly formatting where relevant.
**Below 8 signals:** Wall of text, important details hidden in dense paragraphs, no visual hierarchy, critical dates buried mid-sentence, unformatted lists, impossible to scan quickly.

## Report Template

```
## Communication Evaluation Report

### Message Type
[Email / Slack Message / Proposal / Cover Letter / Announcement / Other]

### Scores

| Category | Score | Status |
|----------|-------|--------|
| Clarity of Purpose | X/10 | PASS/FAIL |
| Tone & Professionalism | X/10 | PASS/FAIL |
| Brevity | X/10 | PASS/FAIL |
| Call to Action | X/10 | PASS/FAIL |
| Audience Awareness | X/10 | PASS/FAIL |
| Formatting & Scannability | X/10 | PASS/FAIL |
| **Overall** | **X/10** | **PASS/FAIL** |

### Issues

#### CRITICAL
- **[Category]** — Problem description
  - Quote: "[exact text]"
  - Why it matters: [impact on the recipient]
  - Fix: [specific rewrite]

#### MAJOR
- **[Category]** — Problem description
  - Quote: "[exact text]"
  - Why it matters: [impact on the recipient]
  - Fix: [specific rewrite]

#### MINOR
- **[Category]** — Problem description
  - Quote: "[exact text]"
  - Suggestion: [specific improvement]

### Improved Version
[Complete rewrite of the message as it should be sent. Preserve the author's intent while fixing all identified issues. This section is MANDATORY.]

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
5. **The Improved Version section is mandatory.** For communication, rewrite the entire message — not just the weak parts. The recipient sees the whole thing, so the improved version must be sendable as-is.

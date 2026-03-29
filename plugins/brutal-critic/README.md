# brutal-critic

Uncompromising quality evaluator for code and text. Auto-detects input type, evaluates against an 8/10 minimum standard, and produces structured PASS/FAIL reports with actionable improvements.

## Agents

### brutal-critic-code-agent

Evaluates code quality. Reads code, researches best practices online (2-3 authoritative sources), and scores across 9 categories:

Correctness, Error Handling, Architecture & Design, Readability & Maintainability, Security, Performance, Testing, Linting & Formatting, API Design (when applicable)

Delivers a PASS/FAIL verdict with cited sources and required fixes.

### brutal-critic-text-agent

Evaluates any non-code content. Auto-detects text type and loads the matching evaluation framework:

- **Technical writing** (RFCs, ADRs, READMEs, architecture docs, diagrams)
- **General writing** (articles, blog posts, essays, general prose)
- **Communication** (emails, Slack messages, proposals, cover letters)
- **Thought/idea** (arguments, reasoning, strategies, decisions)

Delivers a PASS/FAIL verdict plus an improved rewrite of weak sections.

## Skills

| Skill | Categories | Used By |
|-------|-----------|---------|
| `brutal-evaluation-code` | 9 categories | brutal-critic-code-agent |
| `brutal-evaluation-technical-writing` | 7 categories | brutal-critic-text-agent |
| `brutal-evaluation-writing` | 6 categories | brutal-critic-text-agent |
| `brutal-evaluation-communication` | 6 categories | brutal-critic-text-agent |
| `brutal-evaluation-thought` | 6 categories | brutal-critic-text-agent |

All skills share: 1-10 rating scale, 8/10 minimum pass, weakest-link scoring, CRITICAL caps at 5/10.

Skills are loaded automatically by the agents — users never invoke them directly.

## Installation

If installed as part of the `websoftware-tools` marketplace, the plugin is available automatically.

To use standalone:
```bash
claude plugin add /absolute/path/to/bartek-marketplace/plugins/brutal-critic
```

## Usage

### Code evaluation

```
Review the authentication module in src/auth/
Evaluate whether our error handling is production-ready
Run a quality gate check on the payment service before we ship
```

### Text evaluation

```
Review this ADR I wrote for the migration to microservices
Is this email to the client professional enough?
Tear apart this article I wrote about distributed systems
Check if this text reads well in English
Is my reasoning solid on switching to GraphQL?
Evaluate these architecture diagrams for the payment system
```

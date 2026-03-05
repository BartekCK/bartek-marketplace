# brutal-critic

Uncompromising code quality evaluator for Claude Code. Searches best practices online, evaluates against an 8/10 minimum standard, and produces structured PASS/FAIL reports.

## Components

### Agent: brutal-critic-agent

A rational, data-driven evaluator that:
1. Reads all relevant code thoroughly
2. Researches best practices online (2-3 authoritative sources)
3. Evaluates across 8 categories (Correctness, Error Handling, Architecture, Readability, Security, Performance, Testing, API Design)
4. Delivers a PASS/FAIL verdict with cited sources and required fixes

No agent memory — every evaluation starts fresh to prevent accumulated bias.

### Skill: brutal-evaluation

The evaluation framework defining:
- **Rating scale**: 1-10 with explicit meaning for each score
- **8 evaluation categories** with "8+ requires" and "below 8 signals" criteria
- **Report template**: Scores table, issues by severity (CRITICAL/MAJOR/MINOR), references, positives, verdict
- **Scoring rules**: Overall = weakest link, CRITICAL caps category at 5/10, when in doubt score lower

## Installation

```bash
claude plugin add /path/to/plugins/brutal-critic
```

## Usage

Trigger the agent by asking for code evaluation:

```
Review the authentication module in src/auth/
Evaluate whether our error handling is production-ready
Run a quality gate check on the payment service before we ship
Be brutally honest — is this API design any good?
```

# code-researcher

Codebase research and pattern discovery — investigate code patterns, locate references, find similar implementations, and provide placement guidance for new code.

## Components

- **code-researcher-agent** — Autonomous agent for multi-step codebase investigation and synthesis
- **code-search** — Skill with reference material for modern CLI search tools (ripgrep, fd, fzf)
- **git-search** — Skill with reference material for git-based search (cross-branch grep, pickaxe, blame, bisect)

## Usage

The code-researcher-agent triggers when you ask for codebase investigation tasks — finding references, locating patterns, tracing implementations, or discovering where to place new code. It uses the code-search and git-search skills as reference material for constructing search pipelines.

## Prerequisites

The search recipes and reference material assume the following CLI tools are installed:

- **rg** (ripgrep) — content search
- **fd** (fd-find) — filename/path search
- **fzf** — interactive fuzzy filtering
- **bat** — syntax-highlighted file viewing (used in fzf preview panes)

---
name: code-search
description: Reference material for modern CLI search tools (ripgrep, fd, fzf) and their composition patterns. Loaded by the code-researcher-agent for advanced search pipelines. Also triggers when the user explicitly asks about rg, fd, or fzf usage patterns. Note -- for simple searches, use Claude Code's built-in Grep and Glob tools directly.
user-invocable: false
disable-model-invocation: true
---

# Code Search

## 1. Purpose

Modern codebase search is built on a "Holy Trinity" of Rust-based CLI tools: **ripgrep (`rg`)** for content search, **`fd`** for filename and path search, and **`fzf`** for interactive fuzzy filtering. These tools replace `grep`, `find`, and manual enumeration with faster, `.gitignore`-aware, ergonomically superior alternatives that compose cleanly via standard pipes. Apply them as defaults in every search workflow — never reach for `grep -r` or `find . -name`.

## 2. The Holy Trinity

### `rg` — ripgrep (Content Search)

Ripgrep is a Rust-based, parallel, line-oriented search tool. It is the correct default for any task involving searching file *contents*.

**Why `rg` over `grep`:**
- 5–50× faster on large codebases through parallelism and SIMD
- Automatically skips `.git/`, `node_modules/`, and anything in `.gitignore`
- Handles encoding edge cases (UTF-8, UTF-16, binary) gracefully
- Produces structured, colorized output; clean for piping to `fzf`
- Built-in file-type aliases (`-t ts`, `-t py`, `-t rust`) eliminate glob boilerplate

**Core patterns:**

```bash
# Search all file contents for a string
rg "createUser"

# Case-insensitive
rg -i "usercontroller"

# Restrict to a file type
rg -t ts "useEffect"
rg -t py "def authenticate"

# Show 3 lines of context around each match
rg -C 3 "throw new Error"

# List only filenames with matches
rg -l "TODO"

# Word-boundary match (avoids partial matches)
rg -w "init"

# Include hidden files and dotfiles
rg --hidden "API_KEY"

# Flat output ideal for piping to fzf
rg --line-number --no-heading --color=always "pattern"

# Count matches per file
rg -c "console.log"

# Restrict by glob pattern
rg -g "*.config.*" "baseUrl"

# Multiline match
rg -U "describe\(.*\{[\s\S]*?\}\)"
```

**Key flags:**

| Flag | Meaning |
|------|---------|
| `-i` | Case-insensitive |
| `-w` | Word boundaries |
| `-l` | List filenames only |
| `-c` | Count matches |
| `-n` | Show line numbers |
| `--no-heading` | Flat `file:line:match` format for piping |
| `-t TYPE` | Filter by built-in type alias |
| `-g GLOB` | Filter by glob |
| `-C N` | N lines of context |
| `--hidden` | Include dotfiles |
| `-I` | Ignore `.gitignore` rules |
| `-U` | Multiline mode |

**Never use `grep -r`. Always use `rg`.**

---

### `fd` — fd-find (Filename/Path Search)

`fd` is the ergonomic replacement for `find`. Use it whenever the search involves *filenames, paths, or directory structure* rather than file contents.

**Why `fd` over `find`:**
- Syntax is 3–10× shorter for common operations
- Automatic `.gitignore` support
- Sensible defaults: excludes hidden files unless asked
- Built-in parallel execution with `-x` / `-X`
- Smart regex by default, glob with `-g`

**Core patterns:**

```bash
# Find any file matching a pattern (regex)
fd "UserService"

# Find by exact extension
fd -e ts
fd -e md

# Glob-style pattern
fd -g "*.test.ts"

# Files only (exclude directories)
fd -t f "config"

# Directories only
fd -t d "components"

# Include hidden files
fd -H ".env"

# Disable .gitignore filtering
fd -I "vendor"

# Limit search depth
fd -d 2 "index"

# Execute wc -l on each result
fd -e ts -x wc -l

# Execute once with all results (batch)
fd -e json -X cat

# Find and open all matches in editor
fd "Controller" -x code {}
```

**Key flags:**

| Flag | Meaning |
|------|---------|
| `-t f/d/l/x` | File / directory / symlink / executable |
| `-e EXT` | Extension filter |
| `-g PAT` | Glob pattern (vs. default regex) |
| `-H` | Include hidden files |
| `-I` | No `.gitignore` filtering |
| `-d N` | Max depth |
| `-x CMD` | Execute command per result |
| `-X CMD` | Execute once with all results |
| `--exclude PAT` | Exclude matching paths |

**Never use `find . -name "*.ts"`. Use `fd -e ts` instead.**

---

### `fzf` — Fuzzy Finder (Interactive Selection Layer)

`fzf` is a general-purpose interactive fuzzy filter. It is not a standalone search tool — it is a *selection layer* that makes any list interactive. Its greatest power comes from piping `rg` or `fd` output into it.

**When to use `fzf`:**
- Results need human selection before an action is taken
- Building interactive search-and-open workflows
- Connecting search output to an editor, clipboard, or shell command
- Running live searches that re-execute `rg` on every keystroke

**Core patterns:**

```bash
# Basic file picker (uses fd internally if available)
fzf

# Pipe fd output through fzf
fd -t f | fzf

# Preview files with bat while browsing
fd -t f | fzf --preview 'bat --color=always --style=numbers {}'

# Pipe rg results into fzf with file preview
rg --line-number --no-heading --color=always "pattern" \
  | fzf --ansi \
        --preview 'bat --color=always --highlight-line {2} {1}' \
        --delimiter ':'

# Live re-running rg on every keystroke
fzf --bind 'change:reload:rg --no-heading --line-number --color=always {q} 2>/dev/null || true' \
    --ansi --disabled --query "" \
    --preview 'bat --color=always --highlight-line {2} {1}' \
    --delimiter ':'

# Multi-select with Tab, open all in editor
fd -t f | fzf --multi | xargs code

# Select git branch interactively
git branch | fzf | xargs git checkout
```

**Key flags:**

| Flag | Meaning |
|------|---------|
| `--preview 'cmd {}'` | Show preview pane |
| `--ansi` | Parse ANSI color codes from piped input |
| `--delimiter CHAR` | Split fields for `{1}`, `{2}` references |
| `--bind 'event:action'` | Bind keystrokes or events (e.g., `change:reload`) |
| `-m` / `--multi` | Multi-select with Tab |
| `--height 40%` | Inline mode instead of fullscreen |
| `--disabled` | Disable fuzzy matching (for live rg reload) |
| `--query "str"` | Pre-fill the query string |

---

## 3. Supporting Tools

### `bat` — Syntax-Highlighted Cat

`bat` is a `cat` replacement with syntax highlighting, line numbers, and Git diff markers. Use it whenever viewing file contents in the terminal, and always in `fzf` preview panes.

```bash
bat src/index.ts                          # view with syntax highlighting
bat --style=numbers,changes file.ts       # line numbers + git diff markers
bat --color=always file.ts | head -20     # force color when piping
bat --highlight-line 42 file.ts           # highlight a specific line
```

### `zoxide` — Smart `cd`

`zoxide` learns frequently-visited directories and allows jumping to them with partial names. It replaces manual `cd` navigation for any directory visited more than once.

```bash
z claude-plugins      # jump to the most-visited dir matching "claude-plugins"
z src comp            # multi-token: most-visited dir matching both tokens
zi                    # interactive selection via fzf
```

### `yazi` — TUI File Manager

`yazi` is an async Rust terminal file manager for visually navigating large directory trees, previewing files, and batch-selecting entries when scripted output is insufficient.

```bash
yazi                  # open in current directory
yazi /path/to/dir     # open at a specific path
```

---

## 4. Decision Tree

Apply this table to choose the right tool immediately:

| Goal | Tool |
|------|------|
| Find text/pattern inside file contents | `rg "pattern"` |
| Find files by name, extension, or path | `fd "name"` or `fd -e ext` |
| Interactively select from search results | pipe into `fzf` |
| View a file with syntax highlighting | `bat file` |
| Jump to a frequently-used directory | `z name` |
| Navigate a directory tree visually | `yazi` |
| Find content AND interactively open result | `rg ... \| fzf --preview 'bat ...'` |
| Live re-run search on every keystroke | `fzf --bind change:reload:rg ...` |
| Find files AND execute action on each | `fd -x cmd {}` |
| Open search results in editor | `fd \| fzf -m \| xargs $EDITOR` |

**Composition rule:** Start with `rg` (content) or `fd` (filenames). Add `fzf` when interactive selection is needed. Use `bat` in preview panes. Never fall back to `grep -r` or `find . -name`.

**When all three combine:** The most powerful pattern is `rg` feeding structured `file:line:content` output into `fzf` with `bat` rendering the preview pane. This gives instant codebase-wide content search with keyboard-driven navigation and syntax-highlighted context in a single terminal session — no IDE required.

---

## 5. References

For complete installation instructions, exhaustive flag references, and ready-to-run pipeline recipes, consult the bundled reference files:

- **`references/modern-tools.md`** — Full per-tool reference: installation, core syntax, essential flags, and common patterns for rg, fd, fzf, bat, zoxide, and yazi.

- **`references/recipes.md`** — Copy-pasteable shell recipes: the Ultimate Search & Preview pipeline, Interactive Global Content Search, live rg search with keystroke-driven reload, 7+ additional practical recipes, and a graceful degradation table mapping modern tools to POSIX fallbacks.

Load these files when complete flag documentation or pipeline recipes are needed.

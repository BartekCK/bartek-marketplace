# Code Search Recipes

Copy-pasteable shell recipes for common search workflows. Each recipe includes inline comments.

---

## Ultimate Search & Preview

Browse all files in the project with a live syntax-highlighted preview pane.

```bash
fd --type f \
  | fzf \
      --preview 'bat --color=always --style=numbers {}' \
      --preview-window 'right:60%:wrap'
```

- `fd --type f`: list all non-ignored files
- `fzf --preview`: open a preview pane running `bat` on the selected file
- Press Enter to print the selected path (pipe to `xargs $EDITOR` to open directly)

**Open selection in editor:**

```bash
fd --type f \
  | fzf --preview 'bat --color=always --style=numbers {}' \
  | xargs -r ${EDITOR:-vim}
```

---

## Interactive Global Content Search

Search all file contents for a term and interactively pick a match, with file preview jumping to the matched line.

```bash
rg --line-number --no-heading --color=always "TERM" \
  | fzf \
      --ansi \
      --delimiter ':' \
      --preview 'bat --color=always --style=numbers --highlight-line {2} {1}' \
      --preview-window 'right:60%:+{2}+3/3:wrap'
```

- `{1}`: filename field (split by `:`)
- `{2}`: line number field
- `+{2}+3/3`: scroll preview to the matched line
- Replace `"TERM"` with your search string or a shell variable

---

## Live rg Search — Re-runs on Every Keystroke

A fully interactive search that re-executes `rg` as you type, with a live preview.

```bash
fzf \
  --bind 'change:reload:rg --line-number --no-heading --color=always {q} 2>/dev/null || true' \
  --ansi \
  --disabled \
  --query "" \
  --delimiter ':' \
  --preview 'bat --color=always --highlight-line {2} {1}' \
  --preview-window 'right:60%:+{2}+3/3:wrap' \
  --prompt 'rg> '
```

- `--disabled`: turns off fzf's own fuzzy matching; `rg` does all filtering
- `change:reload`: fires on every keystroke, re-running `rg` with current query `{q}`
- `|| true`: prevents fzf from exiting when `rg` returns no results

---

## Find and Open in Editor

Find files by name, select interactively, open in `$EDITOR`.

```bash
fd --type f --extension ts \
  | fzf --multi --preview 'bat --color=always {}' \
  | xargs -r ${EDITOR:-code}
```

- `--multi`: press Tab to select multiple files
- `-r`: suppress `xargs` if nothing is selected

---

## Git-Modified Files — Pick and Open

Interactively browse files changed in the current git working tree.

```bash
git diff --name-only HEAD \
  | fzf --multi \
        --preview 'git diff HEAD -- {} | bat --color=always -l diff' \
  | xargs -r ${EDITOR:-code}
```

- Shows a colored diff preview for each file
- Replace `HEAD` with a branch name or commit SHA for comparison

---

## Find by Extension — Preview and Select

Quickly filter to a specific file type and browse with preview.

```bash
# TypeScript files
fd -e ts | fzf --preview 'bat --color=always --style=numbers {}'

# Markdown files
fd -e md | fzf --preview 'bat --color=always {}'

# Config files (multiple extensions)
fd -e json -e yaml -e toml | fzf --preview 'bat --color=always {}'
```

---

## Interactive Delete

Find files and delete the selected ones after confirmation.

```bash
fd --type f -g "*.log" \
  | fzf --multi --header "Tab=select, Enter=delete" \
  | xargs -r -p rm
```

- `-p` on `xargs`: prompts before executing (confirmation step)
- Remove `-p` to delete without confirmation

---

## Type-Filtered Content Search

Search only within specific file types and interactively view results.

```bash
# Search only in TypeScript files
rg -t ts --line-number --no-heading --color=always "useState" \
  | fzf --ansi --delimiter ':' \
        --preview 'bat --color=always --highlight-line {2} {1}'

# Exclude test files from search
rg -g '!**/*.test.*' -g '!**/*.spec.*' --line-number "pattern" \
  | fzf --ansi --delimiter ':' \
        --preview 'bat --color=always --highlight-line {2} {1}'
```

---

## Directory Jump — zoxide + fzf

Jump to any known directory interactively.

```bash
# Interactive directory selection (built into zoxide)
zi

# Manual: list all zoxide entries and pick one
zoxide query --list | fzf | xargs -r zoxide add && cd "$(zoxide query --list | fzf)"
```

---

## Find Hidden Config Files

Locate dotfiles and hidden config files (e.g., `.env`, `.eslintrc`).

```bash
# All hidden files in project
fd --hidden --type f -g ".*" \
  | fzf --preview 'bat --color=always {}'

# Specific dotfiles
fd --hidden -g ".env*" | fzf --preview 'bat --color=always {}'
```

---

## Find Largest Files in Project

Identify files consuming the most disk space.

```bash
fd --type f -x du -sh {} \
  | sort -rh \
  | head -20
```

---

## Graceful Degradation

When the Holy Trinity tools are unavailable, fall back to POSIX equivalents. Functionality will be reduced but the workflow still works.

| Modern Tool | POSIX Fallback | Notes |
|-------------|---------------|-------|
| `rg "pattern"` | `grep -rn "pattern" .` | Slower, no `.gitignore` respect, no type filtering |
| `rg -t ts "pattern"` | `grep -rn "pattern" --include="*.ts" .` | Verbose flag syntax |
| `rg -l "pattern"` | `grep -rl "pattern" .` | Available in most grep versions |
| `fd -e ts` | `find . -name "*.ts" -not -path "*/node_modules/*"` | Must manually exclude common dirs |
| `fd -t d "name"` | `find . -type d -name "name"` | Direct equivalent |
| `fd -x cmd {}` | `find . -name "pattern" -exec cmd {} \;` | Slightly different syntax |
| `bat file` | `cat file` | No syntax highlighting or line numbers |
| `bat --color=always file \| head` | `cat file \| head` | No color |
| `fzf` (interactive) | List only, manual selection | No interactivity; script around output |
| `z dirname` | `cd ~/full/path/to/dir` | Must know full path |
| `yazi` | `ls -la` or file manager of choice | No TUI navigation |

**Note:** The degradation table is for environments where tools cannot be installed (e.g., restricted CI, minimal containers). Always prefer the modern tools in development environments.

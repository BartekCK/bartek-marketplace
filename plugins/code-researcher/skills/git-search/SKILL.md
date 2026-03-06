---
name: git-search
description: "Provides reference material for git-based search tools and patterns — cross-branch grep, pickaxe queries, blame, bisect, and history tracing. Use for git history and cross-branch investigations."
user-invocable: false
disable-model-invocation: true
---

# Git Search

## 1. Purpose

Git stores the full history of every file, branch, and commit. This makes it a powerful search engine for questions that go beyond "what does the code look like now?" — questions like "when was this introduced?", "which branch has this?", "who last changed this line?", and "what did this file look like on another branch?". These tools complement `rg`/`fd`/`fzf` (see `code-search` skill) by adding the time and branch dimensions.

## 2. Cross-Branch Search

Search code on any branch without switching to it.

### `git grep` — Content Search Across Branches

`git grep` searches tracked file contents. Unlike `rg`, it can target a specific branch or commit.

```bash
# Search current branch (like rg, but git-aware)
git grep "createUser"

# Search a specific branch
git grep "createUser" feature/auth

# Search multiple branches
git grep "createUser" main feature/auth develop

# Search all branches
git grep "createUser" $(git branch -a --format='%(refname:short)')

# Search all local branches
git grep "createUser" $(git branch --format='%(refname:short)')

# Case-insensitive
git grep -i "usercontroller" main

# Show line numbers
git grep -n "TODO" main

# Show surrounding context (3 lines)
git grep -C 3 "handleError" main

# Word boundary match
git grep -w "init" main

# Extended regex
git grep -E "use(Effect|State|Memo)" main

# Count matches per file
git grep -c "console.log" main

# List filenames only
git grep -l "deprecated" main

# Restrict to file pattern
git grep "createUser" main -- "*.ts"
git grep "createUser" main -- "src/services/"
```

**Key flags:**

| Flag | Meaning |
|------|---------|
| `-n` | Show line numbers |
| `-i` | Case-insensitive |
| `-w` | Word boundaries |
| `-l` | Filenames only |
| `-c` | Count per file |
| `-C N` | Context lines |
| `-E` | Extended regex |
| `-- PATH` | Restrict to path/glob |

### View File on Another Branch

```bash
# Show file contents from another branch
git show feature/auth:src/services/UserService.ts

# Diff a file between two branches
git diff main..feature/auth -- src/services/UserService.ts

# Diff entire directories between branches
git diff main..feature/auth -- src/services/

# List files changed between branches
git diff --name-only main..feature/auth

# List files with change stats
git diff --stat main..feature/auth
```

## 3. History Search (Pickaxe)

Find when code was introduced or removed across the entire git history.

### `git log -S` — String Pickaxe

Finds commits where the **number of occurrences** of a string changed (added or removed).

```bash
# Find when "createUser" was introduced or removed
git log -S "createUser" --oneline

# With patch (show the actual diff)
git log -S "createUser" -p

# Restrict to specific file
git log -S "createUser" -- src/services/UserService.ts

# Restrict to file type
git log -S "createUser" -- "*.ts"

# Search all branches
git log -S "createUser" --all --oneline

# With date range
git log -S "createUser" --after="2024-01-01" --before="2025-01-01"

# With author filter
git log -S "createUser" --author="john"
```

### `git log -G` — Regex Pickaxe

Finds commits where a **line matching the regex** was added or removed. More flexible than `-S` but slower.

```bash
# Find commits that changed lines matching a regex
git log -G "use(Effect|State)" --oneline

# With patch
git log -G "function\s+create\w+" -p

# Combined with path restriction
git log -G "TODO|FIXME|HACK" -- "*.ts" --oneline
```

**`-S` vs `-G`:**

| | `-S` (string) | `-G` (regex) |
|---|---|---|
| Matches | Exact string count change | Line content regex |
| Speed | Faster | Slower |
| Use when | You know the exact string | You need pattern matching |

## 4. Commit Message & Author Search

```bash
# Search commit messages
git log --grep="fix auth" --oneline

# Case-insensitive message search
git log --grep="fix auth" -i --oneline

# Search by author
git log --author="john" --oneline

# Combine: message + author
git log --grep="refactor" --author="john" --oneline

# All matching ANY condition (default is AND)
git log --grep="fix" --grep="auth" --all-match --oneline

# Show commits touching a specific file
git log --oneline -- src/services/UserService.ts

# Show commits with stats
git log --stat --oneline -- src/services/
```

## 5. Blame & Line History

### `git blame` — Line-Level Attribution

```bash
# Who last changed each line
git blame src/services/UserService.ts

# Blame a specific line range
git blame -L 10,20 src/services/UserService.ts

# Blame with email instead of name
git blame -e src/services/UserService.ts

# Ignore whitespace changes
git blame -w src/services/UserService.ts

# Detect code movement within file
git blame -M src/services/UserService.ts

# Detect code movement across files
git blame -C src/services/UserService.ts

# Show original filename (for moved/copied code)
git blame -C -C src/services/UserService.ts
```

### `git log -L` — Line Range History

Track how a specific line range evolved over time.

```bash
# History of lines 10-20 in a file
git log -L 10,20:src/services/UserService.ts

# History of a function (git infers boundaries)
git log -L :createUser:src/services/UserService.ts
```

## 6. Finding Which Branch Contains Something

```bash
# Which branches contain a specific commit
git branch --contains abc1234

# Which remote branches contain a commit
git branch -r --contains abc1234

# Which tags contain a commit
git tag --contains abc1234

# Find merge commit that brought a commit into a branch
git log --merges --ancestry-path abc1234..main --oneline
```

## 7. Tracking File Renames

```bash
# Follow file history across renames
git log --follow --oneline -- src/services/UserService.ts

# Follow with patch
git log --follow -p -- src/services/UserService.ts

# Detect renames in diff
git diff --find-renames main..feature/auth
git diff -M main..feature/auth
```

## 8. Git Bisect — Binary Search for Bugs

Find the exact commit that introduced a bug using binary search.

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark a known good commit
git bisect good abc1234

# Git checks out a middle commit — test and mark
git bisect good   # or: git bisect bad

# Repeat until git identifies the first bad commit
# When done:
git bisect reset

# Automated bisect with a test script
git bisect start HEAD abc1234
git bisect run npm test

# Automated with custom command
git bisect run sh -c 'grep -q "expectedString" src/file.ts'
```

## 9. Decision Tree

| Goal | Command |
|------|---------|
| Search code on another branch | `git grep "pattern" branch` |
| View file on another branch | `git show branch:path/to/file` |
| Diff file between branches | `git diff main..branch -- file` |
| Find when string was introduced | `git log -S "string" --oneline` |
| Find when regex pattern changed | `git log -G "regex" -p` |
| Search commit messages | `git log --grep="text" --oneline` |
| Who last changed a line | `git blame file` |
| History of a function | `git log -L :funcName:file` |
| Track file across renames | `git log --follow -- file` |
| Which branch has a commit | `git branch --contains SHA` |
| Find commit that introduced bug | `git bisect start` + mark good/bad |
| List changed files between branches | `git diff --name-only main..branch` |

## 10. Combining with rg/fd/fzf

Pipe git output through the modern tool stack (see `code-search` skill):

```bash
# Interactive branch selection, then grep on it
git branch | fzf | xargs -I{} git grep "pattern" {}

# Pickaxe results into fzf for interactive browsing
git log -S "createUser" --oneline | fzf --preview 'git show {1}'

# Blame with fzf to explore specific lines
git blame src/file.ts | fzf --preview 'git show {1}'

# Search all branches for a pattern, interactively filter
git branch --format='%(refname:short)' \
  | xargs -I{} git grep -n "pattern" {} \
  | fzf --ansi

# Files changed between branches, filtered interactively
git diff --name-only main..HEAD | fzf --preview 'bat --color=always {}'
```

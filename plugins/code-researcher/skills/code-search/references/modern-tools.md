# Modern Search Tools — Complete Reference

Complete installation and flag reference for the Holy Trinity (rg, fd, fzf) and supporting tools (bat, zoxide, yazi).

---

## rg — ripgrep

### Installation

```bash
# macOS
brew install ripgrep

# Debian/Ubuntu
apt install ripgrep

# Windows (winget)
winget install BurntSushi.ripgrep.MSVC

# Cargo
cargo install ripgrep
```

### Core Syntax

```
rg [OPTIONS] PATTERN [PATH...]
rg [OPTIONS] -e PATTERN... [PATH...]
rg [OPTIONS] -f PATTERNFILE... [PATH...]
```

### Essential Flags

| Flag | Long form | Description |
|------|-----------|-------------|
| `-i` | `--ignore-case` | Case-insensitive matching |
| `-s` | `--case-sensitive` | Force case-sensitive |
| `-S` | `--smart-case` | Case-insensitive if pattern is lowercase |
| `-w` | `--word-regexp` | Only match whole words |
| `-x` | `--line-regexp` | Only match whole lines |
| `-F` | `--fixed-strings` | Treat pattern as literal string, not regex |
| `-l` | `--files-with-matches` | Print filenames only |
| `-L` | `--files-without-match` | Print filenames without match |
| `-c` | `--count` | Count matches per file |
| `-n` | `--line-number` | Show line numbers (default when TTY) |
| `-N` | `--no-line-number` | Suppress line numbers |
| `--no-heading` | | Flat `file:line:match` format (good for piping) |
| `-H` | `--with-filename` | Always show filename |
| `-A N` | `--after-context N` | N lines after each match |
| `-B N` | `--before-context N` | N lines before each match |
| `-C N` | `--context N` | N lines before and after each match |
| `-t TYPE` | `--type TYPE` | Restrict to file type alias |
| `-T TYPE` | `--type-not TYPE` | Exclude file type alias |
| `-g GLOB` | `--glob GLOB` | Include/exclude by glob (prefix `!` to exclude) |
| `--hidden` | | Include hidden (dot) files |
| `-I` | `--no-ignore` | Disable `.gitignore` and other ignore files |
| `-u` | `--unrestricted` | Same as `--no-ignore --hidden` (repeat for binary) |
| `-U` | `--multiline` | Allow `.` to match newlines |
| `-P` | `--pcre2` | Use PCRE2 regex engine (lookahead, lookbehind) |
| `-e PAT` | `--regexp PAT` | Specify multiple patterns |
| `--max-depth N` | | Limit directory recursion depth |
| `-m N` | `--max-count N` | Stop after N matches per file |
| `--color always` | | Force color output even when piping |
| `--json` | | Output in JSON format |

### Common Patterns

```bash
# Search for TypeScript function definitions
rg "export (async )?function \w+" -t ts

# Find all TODO/FIXME comments
rg -i "TODO|FIXME|HACK|XXX"

# Search only in src/, exclude tests
rg "pattern" src/ -g '!**/*.test.*'

# Find imports of a specific module
rg "from ['\"].*lodash"

# Count lines matching a pattern per file
rg -c "console\.(log|warn|error)" --sort=count

# Find large match counts (debug noise)
rg -c "console.log" | sort -t: -k2 -rn | head -20

# Search hidden git config files
rg --hidden -g ".git/**" "url ="

# Get type aliases for filtering
rg --type-list | grep -i java
```

---

## fd — fd-find

### Installation

```bash
# macOS
brew install fd

# Debian/Ubuntu
apt install fd-find   # binary is 'fdfind'; alias: ln -s $(which fdfind) ~/.local/bin/fd

# Windows (winget)
winget install sharkdp.fd

# Cargo
cargo install fd-find
```

### Core Syntax

```
fd [OPTIONS] [PATTERN] [PATH...]
```

Pattern is a regex by default. Use `-g` for glob syntax.

### Essential Flags

| Flag | Long form | Description |
|------|-----------|-------------|
| `-t f` | `--type file` | Files only |
| `-t d` | `--type directory` | Directories only |
| `-t l` | `--type symlink` | Symlinks only |
| `-t x` | `--type executable` | Executables only |
| `-e EXT` | `--extension EXT` | Filter by file extension |
| `-g PAT` | `--glob PAT` | Use glob pattern instead of regex |
| `-H` | `--hidden` | Include hidden files |
| `-I` | `--no-ignore` | Disable `.gitignore` filtering |
| `-u` | `--unrestricted` | Same as `-HI` |
| `-d N` | `--max-depth N` | Limit recursion depth |
| `--min-depth N` | | Minimum recursion depth |
| `-x CMD` | `--exec CMD` | Execute command per result |
| `-X CMD` | `--exec-batch CMD` | Execute once with all results |
| `--exclude PAT` | | Exclude paths matching glob |
| `-s` | `--case-sensitive` | Force case-sensitive |
| `-i` | `--ignore-case` | Force case-insensitive |
| `-p` | `--full-path` | Match pattern against full path |
| `-a` | `--absolute-path` | Output absolute paths |
| `-l` | `--list-details` | Long listing format (like `ls -l`) |
| `--changed-within TIME` | | Files changed within duration (e.g., `2d`, `1h`) |
| `--changed-before TIME` | | Files changed before duration |
| `-S SIZE` | `--size SIZE` | Filter by file size (e.g., `+1m`, `-100k`) |

### Common Patterns

```bash
# Find all TypeScript files
fd -e ts

# Find test files only
fd -g "*.test.ts" -t f

# Find all config files up to depth 3
fd -d 3 "config" -t f

# Find recently modified files (last 24h)
fd --changed-within 24h -t f

# Find large files (over 1MB)
fd -S +1m -t f

# Find and delete node_modules
fd -t d -g "node_modules" -X rm -rf

# Print absolute paths
fd -e ts -a

# Find files matching full path (include dir in match)
fd -p "src/.*Controller"
```

---

## fzf — Fuzzy Finder

### Installation

```bash
# macOS
brew install fzf
# Run install script for shell integration (key bindings, completion)
$(brew --prefix)/opt/fzf/install

# Debian/Ubuntu
apt install fzf

# From GitHub releases
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

### Core Syntax

```
fzf [OPTIONS]
COMMAND | fzf [OPTIONS]
```

### Essential Flags

| Flag | Description |
|------|-------------|
| `--query STR` | Pre-fill initial query |
| `--filter STR` | Non-interactive filter mode |
| `--select-1` | Auto-select if exactly one match |
| `--exit-0` | Exit immediately if no match |
| `-m` / `--multi` | Enable multi-select (Tab/Shift-Tab) |
| `--height N%` | Use N% of terminal height (inline) |
| `--layout reverse` | Top-down layout |
| `--border` | Draw border around fzf |
| `--preview CMD` | Run CMD for each item; `{}` is placeholder |
| `--preview-window POS` | Preview position: right/left/up/down/hidden |
| `--ansi` | Parse ANSI color codes in input |
| `--delimiter CHAR` | Field delimiter (default: whitespace) |
| `--nth N` | Limit fuzzy match to Nth field |
| `--with-nth N` | Display only Nth field |
| `--bind KEY:ACTION` | Bind key or event to action |
| `--disabled` | Disable fuzzy matching (for reload workflows) |
| `--phony` | Synonym for `--disabled` |
| `--expect KEYS` | Print pressed key before selection |
| `--no-sort` | Preserve input order |
| `--tac` | Reverse input order |
| `--color SPEC` | Customize colors |

### Common Bind Actions

| Action | Description |
|--------|-------------|
| `reload:CMD` | Re-run CMD and reload list |
| `change:reload:CMD` | Reload on every keystroke |
| `execute:CMD` | Run CMD without exiting fzf |
| `execute-silent:CMD` | Run CMD silently |
| `toggle-preview` | Show/hide preview pane |
| `preview:CMD` | Change preview command |
| `select-all` | Select all filtered items |
| `deselect-all` | Deselect all |

---

## bat — Syntax-Highlighted Cat

### Installation

```bash
brew install bat
apt install bat          # binary may be 'batcat'; alias as needed
cargo install bat
```

### Essential Flags

| Flag | Description |
|------|-------------|
| `--style COMPONENTS` | Comma-separated: numbers, changes, header, grid, rule, full, plain |
| `--color always/never/auto` | Force color output |
| `--highlight-line N` | Highlight line N |
| `--line-range N:M` | Show only lines N to M |
| `-l LANG` | Force language for syntax highlighting |
| `--plain` / `-p` | No decorations (equivalent to cat) |
| `--pager CMD` | Override pager (use `--pager never` to disable) |
| `--list-languages` | List supported languages |
| `--list-themes` | List available themes |
| `--theme THEME` | Use a specific theme |

### Common Patterns

```bash
bat --color=always --style=numbers {}    # fzf preview pane standard
bat --pager never file.ts                # no pager, full output
bat -l json data.json                    # force JSON highlighting
```

---

## zoxide — Smart cd

### Installation

```bash
brew install zoxide
cargo install zoxide

# Shell integration (add to ~/.zshrc or ~/.bashrc)
eval "$(zoxide init zsh)"     # or bash, fish, etc.
```

### Commands

| Command | Description |
|---------|-------------|
| `z QUERY` | Jump to best matching directory |
| `z QUERY1 QUERY2` | Multi-token match |
| `zi` | Interactive selection via fzf |
| `z -` | Go to previous directory |
| `zoxide add PATH` | Manually add a path |
| `zoxide remove PATH` | Remove a path from database |
| `zoxide query QUERY` | Print best match without changing dir |
| `zoxide query --list` | List all matches with scores |

---

## yazi — TUI File Manager

### Installation

```bash
brew install yazi
cargo install yazi-fm yazi-cli

# Optional enhanced dependencies
brew install ffmpegthumbnailer unar poppler fd ripgrep fzf zoxide
```

### Key Bindings (defaults)

| Key | Action |
|-----|--------|
| `h/j/k/l` or arrows | Navigate |
| `Enter` | Open file / enter directory |
| `q` | Quit |
| `Space` | Toggle selection |
| `y` | Yank (copy) |
| `p` | Paste |
| `d` | Move to trash |
| `r` | Rename |
| `/` | Search (fd-powered) |
| `f` | Filter |
| `s` | Sort menu |
| `z` | Jump via zoxide |

---
name: serena-project-init
description: Guides through the full Serena project registration and initial activation workflow. Triggers on "register a Serena project", "init Serena", "set up Serena for this project", "add project to Serena", "configure Serena", or when encountering "No source files found" errors during project registration.
user-invocable: false
disable-model-invocation: true
---

# Serena Project Initialization

Initialize and activate a Serena project so that semantic coding tools (symbol search, code navigation, file editing, codebase exploration) become available via MCP.

## When to Use

- Setting up Serena for a new codebase
- Encountering "No source files found" or language detection failures during `activate_project`
- Needing to register a project with a specific programming language

## Prerequisites

Serena must be available as an MCP server. Verify the `.mcp.json` configuration includes a Serena entry using `uvx` with `start-mcp-server`.

## Initialization Workflow

### Step 1: Identify the Project Language

Determine the primary programming language(s) of the project. Scan for source files to identify the language if not already known.

Supported languages (common ones):
`python`, `typescript`, `rust`, `go`, `java`, `kotlin`, `ruby`, `swift`, `cpp`, `csharp`, `dart`, `php`, `bash`, `markdown`, `yaml`, `toml`

For the full list, consult `references/supported-languages.md`.

**Important notes:**
- For JavaScript projects, use `typescript`
- For C projects, use `cpp`
- For projects with no traditional source code (markdown/config only), use `markdown`
- Multiple languages can be specified by passing `--language` multiple times

### Step 2: Register the Project via CLI

Run the Serena CLI to create the project configuration:

```bash
uvx --from "git+https://github.com/oraios/serena" serena project create <project-path> --name <project-name> --language <language>
```

Parameters:
- `<project-path>` — absolute path to the project root
- `--name` — human-readable project name (defaults to directory name)
- `--language` — programming language (can be repeated for multi-language projects)
- `--index` — (optional) index the project immediately after creation

This creates a `.serena/project.yml` configuration file in the project directory.

### Step 3: Update .gitignore

Add `.serena` to the project's `.gitignore` to prevent committing Serena's local configuration. If `.gitignore` does not exist, create it.

**Before appending, verify `.serena` is not already listed to avoid duplicates.**

```bash
# Check if .serena is already in .gitignore
grep -qxF ".serena" <project-path>/.gitignore 2>/dev/null || echo ".serena" >> <project-path>/.gitignore
```

### Step 4: Activate the Project

Use the `activate_project` MCP tool to activate the registered project:

```
activate_project(project="<project-path>")
```

Pass either the project path or the registered project name. On success, all Serena semantic tools become available for the project.

### Step 5: Run Onboarding (Optional)

After activation, optionally run onboarding to let Serena analyze the project structure:

```
onboarding()
```

This identifies project structure, build systems, and testing frameworks, storing the results for future reference.

## Troubleshooting

### "No source files found"

The project has no files matching any supported language. Fix by explicitly specifying the language:

```bash
uvx --from "git+https://github.com/oraios/serena" serena project create <path> --language <language>
```

### "No active project"

The `activate_project` tool must be called before using any other Serena tools. The `execute_shell_command` tool also requires an active project. For per-session activation of an already-registered project, use the `serena-project-activate` skill.

### Project Already Registered

If `.serena/project.yml` already exists, `activate_project` will use it directly. To re-register, delete the existing config and run `project create` again.

## Project Configuration

The generated `.serena/project.yml` contains settings for:
- **languages** — language servers to start
- **encoding** — file encoding (default: utf-8)
- **ignored_paths** — paths to exclude (gitignore syntax)
- **read_only** — disable editing tools
- **excluded_tools** / **included_optional_tools** — customize available tools
- **initial_prompt** — prompt given on project activation

For the full configuration reference, consult `references/project-config.md`.

## Additional Resources

### Reference Files

- **`references/supported-languages.md`** — Complete list of supported languages with notes
- **`references/project-config.md`** — Full project.yml configuration reference

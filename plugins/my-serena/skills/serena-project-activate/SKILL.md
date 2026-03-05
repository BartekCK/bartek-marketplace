---
name: serena-project-activate
description: This skill should be used when the user encounters "No active project" errors from Serena, when the user asks to "activate Serena", "start Serena session", or "make Serena work", or when Serena tools are not responding at session start despite the project being already registered.
---

# Serena Project Activation

Activate an already-registered Serena project for the current session so that semantic coding tools become available.

## When to Use

This skill applies when:
- The project already has a `.serena/project.yml` configuration file (already registered)
- `get_current_config` returns "No active project" error
- Serena MCP tools fail or return errors at session start
- The user explicitly asks to activate or start Serena

**If `.serena/project.yml` does not exist**, the project is not yet registered. Use the `serena-project-init` skill instead to perform full registration.

## Why Activation Is Per-Session

Serena separates registration from activation by design:
- **Registration** (via CLI `serena project create`) is persistent — it writes `.serena/project.yml` to disk and only needs to happen once.
- **Activation** (via `activate_project` MCP tool) is an in-memory operation — it must happen each time a new session starts with the Serena MCP server.

This means every new conversation or session requires calling `activate_project` before any other Serena tools will work.

## Activation Workflow

### Step 1: Identify the Project

Determine the project name or path. Options:
- Use the registered project name from `.serena/project.yml` (the `project_name` field)
- Use the absolute path to the project directory
- Call `get_current_config` — it lists known projects by name in its error message

### Step 2: Activate the Project

Call the `activate_project` MCP tool:

```
activate_project(project="<project-name-or-path>")
```

Either the registered project name or the absolute directory path works. On success, all Serena semantic tools become available.

### Step 3: Verify Activation

Call `get_current_config` to confirm the project is now active. A successful response returning configuration data (rather than a "No active project" error) confirms activation succeeded.

### Step 4: Check Onboarding

Call `check_onboarding_performed` to determine whether onboarding data exists for this project. If onboarding has not been performed, consider running `onboarding()` to let Serena analyze the project structure, build systems, and testing frameworks.

## Troubleshooting

### "Project not found" on activate

The project name may not match what was registered. Try:
- Using the absolute path instead of the project name
- Checking `.serena/project.yml` for the exact `project_name` value
- Running `get_current_config` to see the list of known projects

### Activation succeeds but tools still fail

Check that the configured language servers can start:
- Verify the correct language is configured in `.serena/project.yml`
- Ensure language server dependencies are installed (e.g., `pyright` for Python, `typescript-language-server` for TypeScript)

### Multiple projects

Only one project can be active at a time. Activating a new project deactivates the previous one. To switch projects, call `activate_project` with the new project name or path.

### Project needs re-registration

If the `.serena/project.yml` is corrupted or misconfigured, delete it and use the `serena-project-init` skill to register the project again from scratch.

## Cross-Reference

- For initial project registration and setup, use the `serena-project-init` skill
- For full project configuration options, consult the `serena-project-init` skill's `references/project-config.md`

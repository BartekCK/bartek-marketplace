# my-serena

Semantic coding tools via MCP (Model Context Protocol) — symbol search, code navigation, file editing, and codebase exploration for Claude Code.

## Components

- **serena-agent** — Agent for Serena project setup, activation, and troubleshooting
- **serena-project-init** — Skill for first-time Serena project registration and configuration
- **serena-project-activate** — Skill for per-session Serena project activation

## Prerequisites

- [Serena](https://github.com/oraios/serena) installed and accessible via `uvx`. See [Serena installation guide](https://github.com/oraios/serena#quick-start)
- A project with supported source files (Python, TypeScript, Java, etc.)

## MCP Server

This plugin configures the `serena` MCP server via `.mcp.json`, using `uvx` to run the Serena language server for semantic code operations.

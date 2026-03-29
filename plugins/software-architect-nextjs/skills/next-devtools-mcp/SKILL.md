---
name: next-devtools-mcp
description: "This skill provides reference for using the next-devtools-mcp tools — get_errors, get_logs, get_page_metadata, get_project_metadata, get_routes, get_server_action_by_id. Loaded by the nextjs-architect-agent to understand available MCP capabilities."
user-invocable: false
disable-model-invocation: true
---

# Next.js DevTools MCP Reference

Reference guide for the `next-devtools-mcp` MCP server tools. The server has a two-layer architecture: **wrapper tools** that work independently of the dev server, and **runtime tools** that require a running Next.js 16+ development server via the built-in `/_next/mcp` endpoint.

## Prerequisites

- Next.js 16+ project with dev server running (`npm run dev` / `pnpm dev`) — required for runtime tools only
- `next-devtools-mcp` configured in the project's `.mcp.json`
- The MCP server auto-discovers running Next.js instances on localhost

## Wrapper Tools

These tools work independently of the dev server:

### nextjs_docs
Search and retrieve Next.js official documentation and best practices. This is the primary tool for the documentation-first principle — query it for current API details, patterns, and migration guidance.

### nextjs_index
Discover running Next.js dev servers and their available runtime tools. Use at the start of any session to confirm dev server availability.

### browser_eval
Automate and test web applications using Playwright. Use during validation phase to verify page rendering and test interactions.

### nextjs_call
Execute tools on specific Next.js dev server instances. Use when multiple dev servers are running and you need to target a specific one.

### upgrade_nextjs_16
Guided upgrade assistant for migrating to Next.js 16 with codemods.

### enable_cache_components
Setup and migration helper for Cache Components mode.

## Runtime Tools

### get_errors

Retrieve current build errors, runtime errors, and type errors from the dev server.

**When to use**: Before starting work (check current state), after implementation (validate no regressions), when debugging issues.

**Returns**: Error list grouped by type — build errors, runtime errors, type errors. Each error includes file path, line number, and error message.

**Usage pattern**:
```
1. Call get_errors to check baseline before changes
2. Implement changes
3. Call get_errors again to verify no new errors introduced
```

### get_logs

Get the path to the development log file containing browser console logs and server output.

**When to use**: When debugging runtime behavior, checking console warnings, or investigating server-side logging output.

**Returns**: Path to the log file. Read this file to inspect console messages and server output.

### get_page_metadata

Get metadata about specific pages including routes, components, and rendering information.

**When to use**: To verify a page's rendering mode (static vs dynamic), check which components are loaded, inspect metadata configuration, or understand the component hierarchy of a specific route.

**Parameters**: `page` — the route path (e.g., `/dashboard`, `/blog/[slug]`)

**Returns**: Route information, component tree, rendering details, and metadata for the specified page.

### get_project_metadata

Retrieve project structure, configuration, and dev server URL.

**When to use**: At the start of any Next.js task to understand the project setup — framework version, configuration, directory structure, and dev server status.

**Returns**: Project configuration including Next.js version, enabled features, dev server URL, and project structure overview.

### get_routes

Get all routes that become entry points by scanning the filesystem. Returns routes grouped by router type (appRouter, pagesRouter).

**When to use**: To understand the full routing structure, verify new routes are registered, check for route conflicts, or plan where to add new pages.

**Returns**: Routes grouped by router type. Dynamic segments appear as `[param]` or `[...slug]` patterns. Includes information about route handlers, layouts, and page files.

### get_server_action_by_id

Look up Server Actions by their ID to find the source file and function name.

**When to use**: When debugging Server Action behavior, tracing action calls, or verifying that a Server Action is properly registered and accessible.

**Parameters**: `id` — the Server Action ID string

**Returns**: Source file path and function name for the specified Server Action.

## Workflow Patterns

### Initial Project Assessment

```
1. get_project_metadata → understand version, config, structure
2. get_routes → map existing routing
3. get_errors → check current health
```

### Post-Implementation Validation

```
1. get_errors → check for build/runtime/type errors
2. get_routes → verify new routes are registered
3. get_page_metadata → confirm rendering mode and component setup
```

### Debugging

```
1. get_errors → identify error type and location
2. get_logs → check console output and server logs
3. get_page_metadata → inspect component hierarchy
4. get_server_action_by_id → trace action calls if relevant
```

## Important Notes

- All tools require a running Next.js dev server. If the server is not running, tools return connection errors.
- The MCP server auto-discovers Next.js instances on multiple ports — no manual port configuration needed.
- Tool results reflect real-time application state. Re-query after making changes to get updated information.
- The `next-devtools-mcp` server exposes a `nextjs_docs` tool for querying the Next.js knowledge base directly. Prefer MCP tool results over training data for framework-specific details.

# software-architect-nextjs

Next.js architect agent with deep framework expertise, MCP-powered documentation access, and implementation delegation.

## Components

### Agent: `nextjs-architect-agent`

Senior Next.js architect that:

1. **Checks docs first** — uses `next-devtools-mcp` to query documentation and live app state before making decisions
2. **Designs the architecture** — plans file structure, component boundaries, data fetching, caching strategies
3. **Delegates implementation** — splits work across `frontend-software-developer-agent`, `backend-software-developer-agent`, and `tester-agent` via Agent tool
4. **Validates results** — offers post-implementation validation using MCP tools (errors, routes, metadata)

**Model**: Opus (for thorough architectural analysis)

**Triggers on**: Next.js, server components, client components, server actions, routes, app router, middleware, layouts, SSR, SSG, ISR, streaming, caching, hydration, and all Next.js-related concepts.

### Skill: `next-devtools-mcp`

Agent-exclusive reference skill documenting all 6 MCP tools:
- `get_errors` — build, runtime, and type errors
- `get_logs` — dev server and console logs
- `get_page_metadata` — page rendering details
- `get_project_metadata` — project config and structure
- `get_routes` — filesystem route scanning
- `get_server_action_by_id` — server action lookup

### MCP: `next-devtools`

Connects to running Next.js 16+ dev servers via the built-in `/_next/mcp` endpoint.

## Prerequisites

- Next.js 16+ project with dev server running. If using Next.js 15 or earlier, MCP runtime tools will be unavailable — the agent falls back to training knowledge only.
- `software-development` plugin installed (provides implementation agents). Install from `plugins/software-development/` in this marketplace.

## Usage

The agent activates automatically when Next.js topics are discussed. It follows a strict workflow:

1. Research via MCP tools
2. Design architecture
3. Present plan for approval
4. Delegate to software-developer agents
5. Offer validation (with user consent)

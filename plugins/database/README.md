# database

Database inspection and querying via dbhub MCP — explore schemas, run SQL queries, write migrations, and connect to databases.

## Components

- **database-agent** — Autonomous agent for database inspection, querying, and schema exploration
- **db-connect** — Skill for establishing database connections mid-session
- **.mcp.json** — dbhub MCP server configuration (requires `DATABASE_URL` environment variable)

## Setup

Set `DATABASE_URL` in your shell environment **before** starting Claude Code so the dbhub MCP server can start automatically:

```bash
export DATABASE_URL=postgres://user:password@localhost:5432/dbname
```

Add this to your `.zshrc` or `.bashrc` to persist it across sessions.

The dbhub MCP server reads `DATABASE_URL` at startup and exposes database tools to the agent. If the variable is not set when Claude Code launches, the **db-connect** skill provides a fallback that discovers credentials from project files and uses native CLI tools (`psql`, `mysql`, `sqlite3`, `sqlcmd`) for the remainder of the session.

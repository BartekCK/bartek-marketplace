---
name: db-connect
description: Establishes a database connection mid-session when DATABASE_URL was not set at startup. Used internally by the database-agent when dbhub MCP tools are unavailable. Searches project files for credentials, asks the user interactively if needed, and falls back to native CLI tools (psql, mysql, sqlite3, sqlcmd) for the remainder of the session.
user-invocable: false
---

# Database Connection Setup

Establishes a database connection when `DATABASE_URL` was not set at session start, allowing database operations to proceed without restarting Claude Code. Since dbhub MCP tools require `DATABASE_URL` at session load, this skill uses native database CLI tools (`psql`, `mysql`, `sqlite3`, `sqlcmd`) for the current session.

**Steps 1–3 find the DSN. Steps 4–6 apply it and verify connectivity.**

---

## Step 1: Check Shell Environment

Check if `DATABASE_URL` is already set in the current shell:

```bash
echo $DATABASE_URL
```

If non-empty, skip to Step 4.

---

## Step 2: Search Project for Credentials

Use Glob to find environment files:

```
**/.env
**/.env.local
**/.env.development
**/.env.development.local
**/docker-compose.yml
**/docker-compose.yaml
```

Read each file found. Look for (case-insensitive):
- `DATABASE_URL`, `DB_URL`, `DB_DSN`, `DB_CONNECTION_STRING`
- `POSTGRES_URL`, `POSTGRESQL_URL`
- `MYSQL_URL`, `MSSQL_URL`
- `DATABASE_HOST` + `DATABASE_PORT` + `DATABASE_NAME` + `DATABASE_USER` + `DATABASE_PASSWORD`

For `docker-compose.yml`, look inside `postgres`, `mysql`, `mariadb`, or `mssql` service definitions under `environment:`.

Assemble DSN from found values:
- PostgreSQL: `postgres://USER:PASSWORD@HOST:PORT/DBNAME?sslmode=disable`
- MySQL/MariaDB: `mysql://USER:PASSWORD@HOST:PORT/DBNAME`
- SQL Server: `sqlserver://USER:PASSWORD@HOST:PORT/DBNAME`
- SQLite: the file path (e.g. `/path/to/db.sqlite`)

---

## Step 3: Collect Credentials Interactively

If no credentials are found in project files, ask the user:

> "I couldn't find database connection details in your project files. Please provide:
> - Database type (PostgreSQL, MySQL/MariaDB, SQL Server, SQLite)
> - Host, port, database name, username, password
>
> Or paste a full connection string if you have one."

---

## Step 4: Store the DSN

**Do not use `export`** — each Bash tool invocation runs in an isolated subshell, so exported variables do not persist across calls. Embed the full DSN string directly into every subsequent CLI command. Example: `psql "postgres://user:pass@host:5432/db" -c "SELECT 1;"`

---

## Step 5: Verify Connectivity

Confirm the DSN is reachable before proceeding to native CLI queries. Pass the DSN directly to the appropriate CLI tool:

```bash
# PostgreSQL — pass DSN as connection string argument
psql "postgres://USER:PASSWORD@HOST:PORT/DBNAME" -c "SELECT 1;" && echo "Connected"

# MySQL — the mysql CLI does not accept DSN URLs, pass components as flags
mysql -h <HOST> -u <USER> -p<PASSWORD> <DBNAME> -e "SELECT 1;"

# SQLite — file existence is the connectivity check
ls -lh /path/to/db.sqlite

# SQL Server — pass connection parameters as flags
sqlcmd -S HOST,PORT -U USER -P PASSWORD -d DBNAME -Q "SELECT 1;"
```

If the connection check fails, report the exact error to the user and ask them to verify the credentials before continuing.

---

## Step 6: Confirm Connection Mode

Inform the user:

> "Connected. For this session I'll use native database tools (psql/mysql/sqlite3/sqlcmd) since the dbhub MCP server wasn't running at session start. Next session, add `DATABASE_URL=<dsn>` to your `.env` file so dbhub starts automatically."

Proceed with database operations using native CLI commands.

---

## Reference Material

For native CLI command syntax (psql, mysql, sqlite3, sqlcmd), schema exploration queries, and instructions for persisting credentials to avoid this issue in future sessions, load **`references/native-cli.md`**.

---
name: Database Connection Setup
description: This skill should be used when the user forgot to set DATABASE_URL before starting their Claude Code session, when the database agent cannot connect to the database, when dbhub MCP tools are unavailable, when the user says "database not connected", "can't reach the database", or asks to "connect to database mid-session", "set up database credentials", "configure database connection", or "fix database connection without restarting". Establishes a working database connection during an active session without restarting Claude Code.
version: 1.0.0
---

# Database Connection Setup

Establishes a database connection when `DATABASE_URL` was not set at session start, allowing database operations to proceed without restarting Claude Code. Since dbhub MCP tools require `DATABASE_URL` at session load, this skill uses native database CLI tools (`psql`, `mysql`, `sqlite3`, `mongosh`) for the current session.

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
- `MYSQL_URL`, `MONGODB_URI`
- `DATABASE_HOST` + `DATABASE_PORT` + `DATABASE_NAME` + `DATABASE_USER` + `DATABASE_PASSWORD`

For `docker-compose.yml`, look inside `postgres`, `mysql`, or `mongo` service definitions under `environment:`.

Assemble DSN from found values:
- PostgreSQL: `postgres://USER:PASSWORD@HOST:PORT/DBNAME?sslmode=disable`
- MySQL: `mysql://USER:PASSWORD@HOST:PORT/DBNAME`
- SQLite: the file path (e.g. `/path/to/db.sqlite`)
- MongoDB: `mongodb://USER:PASSWORD@HOST:PORT/DBNAME`

---

## Step 3: Collect Credentials Interactively

If no credentials are found in project files, ask the user:

> "I couldn't find database connection details in your project files. Please provide:
> - Database type (PostgreSQL, MySQL, SQLite, MongoDB)
> - Host, port, database name, username, password
>
> Or paste a full connection string if you have one."

---

## Step 4: Export DATABASE_URL

Export the assembled DSN to the current shell:

```bash
export DATABASE_URL="<assembled-dsn>"
```

---

## Step 5: Verify Connectivity

Confirm the DSN is reachable before proceeding to native CLI queries. Use the appropriate tool for the database type:

```bash
# PostgreSQL
psql "$DATABASE_URL" -c "SELECT 1;" && echo "Connected"

# MySQL — the mysql CLI does not accept DSN URLs, parse components separately
# (see references/native-cli.md for the flag syntax)
mysql -h HOST -u USER -pPASSWORD DBNAME -e "SELECT 1;"

# SQLite — file existence is the connectivity check
ls -lh /path/to/db.sqlite

# MongoDB
mongosh "$DATABASE_URL" --eval "db.runCommand({ping: 1})"
```

If the connection check fails, report the exact error to the user and ask them to verify the credentials before continuing.

---

## Step 6: Confirm Connection Mode

Inform the user:

> "Connected. For this session I'll use native database tools (psql/mysql/sqlite3) since the dbhub MCP server wasn't running at session start. Next session, add `DATABASE_URL=<dsn>` to your `.env` file so dbhub starts automatically."

Proceed with database operations using native CLI commands.

---

## Reference Material

For native CLI command syntax (psql, mysql, sqlite3, mongosh), schema exploration queries, and instructions for persisting credentials to avoid this issue in future sessions, load **`references/native-cli.md`**.

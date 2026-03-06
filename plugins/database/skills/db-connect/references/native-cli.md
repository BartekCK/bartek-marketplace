# Native CLI Database Commands

Reference for executing database queries and exploring schemas when dbhub MCP tools are unavailable. Use via `Bash` tool.

---

## PostgreSQL (psql)

```bash
# Test connection
psql "DSN" -c "SELECT version();"

# List all tables
psql "DSN" -c "\dt"

# Run a single query
psql "DSN" -c "SELECT * FROM table_name LIMIT 10;"

# Multi-line query using heredoc
psql "DSN" << 'EOF'
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'users'
ORDER BY ordinal_position;
EOF

# Tables with row counts
psql "DSN" -c "
SELECT schemaname, tablename, n_live_tup AS row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;"

# Foreign keys for a table
psql "DSN" << 'EOF'
SELECT
  kcu.column_name,
  ccu.table_name AS foreign_table,
  ccu.column_name AS foreign_column
FROM information_schema.table_constraints AS tc
JOIN information_schema.key_column_usage AS kcu
  ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu
  ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND tc.table_name = 'target_table';
EOF

# List indexes
psql "DSN" -c "
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'target_table';"
```

---

## MySQL

The `mysql` CLI does not accept DSN URL format. Parse the DSN components (`mysql://USER:PASSWORD@HOST:PORT/DBNAME`) and pass them as discrete flags:

```bash
# DSN: mysql://alice:PASSWORD@localhost:3306/myapp
mysql -h localhost -u alice -pPASSWORD myapp -e "SHOW TABLES;"

# Run a query
mysql -h HOST -u USER -pPASSWORD DBNAME -e "SELECT * FROM table_name LIMIT 10;"

# Tables with approximate row counts
mysql -h HOST -u USER -pPASSWORD DBNAME -e "
SELECT table_name, table_rows
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY table_rows DESC;"

# Describe a table
mysql -h HOST -u USER -pPASSWORD DBNAME -e "DESCRIBE table_name;"
```

---

## SQLite

```bash
# List all tables
sqlite3 /path/to/db.sqlite ".tables"

# Describe a table schema
sqlite3 /path/to/db.sqlite ".schema table_name"

# Run a query
sqlite3 /path/to/db.sqlite "SELECT * FROM table_name LIMIT 10;"

# Row counts for all tables (generate dynamically)
sqlite3 /path/to/db.sqlite "
SELECT name FROM sqlite_master WHERE type='table';" | while read tbl; do
  echo -n "$tbl: "
  sqlite3 /path/to/db.sqlite "SELECT COUNT(*) FROM \"$tbl\";"
done
```

---

## SQL Server (sqlcmd)

```bash
# Test connection
sqlcmd -S HOST,PORT -U USER -P PASSWORD -d DBNAME -Q "SELECT 1;"

# List all tables
sqlcmd -S HOST,PORT -U USER -P PASSWORD -d DBNAME -Q "SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE';"

# Run a query
sqlcmd -S HOST,PORT -U USER -P PASSWORD -d DBNAME -Q "SELECT TOP 10 * FROM table_name;"

# Describe a table
sqlcmd -S HOST,PORT -U USER -P PASSWORD -d DBNAME -Q "SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'table_name';"

# Tables with row counts
sqlcmd -S HOST,PORT -U USER -P PASSWORD -d DBNAME -Q "SELECT t.NAME, p.rows FROM sys.tables t JOIN sys.partitions p ON t.object_id = p.object_id WHERE p.index_id < 2 ORDER BY p.rows DESC;"
```

---

## Preventing Future Connection Issues

To avoid needing this skill again, persist credentials so dbhub starts automatically next session.

**Option A — Project `.env` file:**

```
DATABASE_URL=postgres://user:password@localhost:5432/dbname?sslmode=disable
```

Make sure `.env` is listed in `.gitignore` before saving credentials there.

**Option B — Shell profile:**

```bash
echo 'export DATABASE_URL=postgres://user:password@localhost:5432/dbname' >> ~/.zshrc
source ~/.zshrc
```

After either option, restart Claude Code. The plugin's `.mcp.json` will start dbhub automatically using `${DATABASE_URL}`, making MCP tools available for the full session.

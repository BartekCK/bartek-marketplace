# Native CLI Database Commands

Reference for executing database queries and exploring schemas when dbhub MCP tools are unavailable. Use via `Bash` tool.

---

## PostgreSQL (psql)

```bash
# Test connection
psql "$DATABASE_URL" -c "SELECT version();"

# List all tables
psql "$DATABASE_URL" -c "\dt"

# Run a single query
psql "$DATABASE_URL" -c "SELECT * FROM table_name LIMIT 10;"

# Multi-line query using heredoc
psql "$DATABASE_URL" << 'EOF'
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'users'
ORDER BY ordinal_position;
EOF

# Tables with row counts
psql "$DATABASE_URL" -c "
SELECT schemaname, tablename, n_live_tup AS row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;"

# Foreign keys for a table
psql "$DATABASE_URL" << 'EOF'
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
psql "$DATABASE_URL" -c "
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'target_table';"
```

---

## MySQL

The `mysql` CLI does not accept DSN URL format. Parse the DSN components (`mysql://USER:PASSWORD@HOST:PORT/DBNAME`) and pass them as discrete flags:

```bash
# DSN: mysql://alice:secret@localhost:3306/myapp
mysql -h localhost -u alice -psecret myapp -e "SHOW TABLES;"

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
  sqlite3 /path/to/db.sqlite "SELECT COUNT(*) FROM $tbl;"
done
```

---

## MongoDB (mongosh)

```bash
# List collections
mongosh "$DATABASE_URL" --eval "db.getCollectionNames()"

# Sample one document
mongosh "$DATABASE_URL" --eval "db.collection_name.findOne()"

# Count documents per collection
mongosh "$DATABASE_URL" --eval "
db.getCollectionNames().forEach(c => print(c + ': ' + db[c].countDocuments()))
"

# Query with filter
mongosh "$DATABASE_URL" --eval "
db.users.find({ created_at: { \$gte: new Date(Date.now() - 30*24*3600*1000) } }).limit(10).toArray()
"
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

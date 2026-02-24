---
name: db-local
description: Local dev database management (psql, schema, reset, seed, dump, restore)
allowed-tools: Bash(PGPASSWORD=*), Bash(psql *), Bash(pg_dump *), Bash(docker inspect *), Bash(lsof *), Bash(kill *), Bash(sleep *), Bash(DATABASE_URL=*), AskUserQuestion
argument-hint: "[shell|schema|tables|reset|seed|dump [file]|restore <file>|query <SQL>]"
---

Manage the local development PostgreSQL database.

**Local DB constants:**
- Port: `6432`
- User: `postgres`
- Password: `123456`
- Database: `accounting`
- Container: `accounting-db`

## Behavior

Arguments: $ARGUMENTS

**Step 1: Verify local DB is running**

```bash
docker inspect accounting-db --format '{{.State.Status}}' 2>/dev/null
```

If the container is not running, tell the user to start it with `/flash:dev` first.

**Step 2: Execute subcommand**

Connection: `PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting`

Parse `$ARGUMENTS`:

- **(none)** or **`shell`** → open interactive psql session:
  ```bash
  PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting
  ```

- **`schema`** → show all tables and column details:
  ```bash
  PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting -c "\dt+"
  ```
  Then for each table found:
  ```bash
  PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting -c "\d+ <table>"
  ```

- **`tables`** → list tables with row counts:
  ```bash
  PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting -c "SELECT schemaname, tablename, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC;"
  ```

- **`reset`** → **DESTRUCTIVE: Confirm with user first using AskUserQuestion!**
  1. Ask: "This will DROP and recreate the accounting database. All data will be lost. Continue?"
  2. If confirmed:
     ```bash
     PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -c "DROP DATABASE IF EXISTS accounting;"
     PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -c "CREATE DATABASE accounting;"
     ```
  3. Find and kill the running Go server (if any):
     ```bash
     lsof -i :8080 -sTCP:LISTEN -t 2>/dev/null | xargs kill 2>/dev/null || true
     ```
  4. Tell the user to restart with `/flash:dev` — GORM auto-migrates and seeds on startup.

- **`seed`** → restart server with seed flag:
  1. Kill existing server:
     ```bash
     lsof -i :8080 -sTCP:LISTEN -t 2>/dev/null | xargs kill 2>/dev/null || true
     ```
  2. Tell the user to start the server with `SEED_ON_EMPTY=true` via `/flash:dev`.

- **`dump [file]`** → export database:
  ```bash
  pg_dump -h localhost -p 6432 -U postgres -d accounting > <file>
  ```
  If no file specified, use a timestamped name: `accounting_dump_YYYYMMDD_HHMMSS.sql`

- **`restore <file>`** → **DESTRUCTIVE: Confirm with user first using AskUserQuestion!**
  1. Ask: "This will overwrite the accounting database with the dump from <file>. Continue?"
  2. If confirmed:
     ```bash
     PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -c "DROP DATABASE IF EXISTS accounting;"
     PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -c "CREATE DATABASE accounting;"
     PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting < <file>
     ```
  3. Report success and row counts.

- **`query <SQL>`** → run a one-off query:
  ```bash
  PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting -c "<SQL>"
  ```

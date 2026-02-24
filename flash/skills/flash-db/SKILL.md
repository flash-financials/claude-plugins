---
name: flash-db
description: Flash Financials database operations — connect to local PostgreSQL, view schema and tables, reset or seed database, dump and restore, run queries, access production database via SSH tunnel.
---

# Flash Financials Database

## Context

- PostgreSQL: !`docker inspect -f '{{.State.Status}}' accounting-db 2>/dev/null || echo "not running"`
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`

## Environment Constants

- DB port: `6432`, user: `postgres`, password: `123456`, database: `accounting`
- Container: `accounting-db`
- Production tunnel: port `5433` (via SSH tunnel through VPS)

---

## Local Database

Connection: `PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting`

Operations:
- `schema` -> `\dt+` then `\d+ <table>` for each
- `tables` -> `SELECT schemaname, tablename, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC`
- `reset` -> **CONFIRM FIRST** -> DROP + CREATE database, kill server, restart with `/flash:flash`
- `seed` -> kill server, restart with `SEED_ON_EMPTY=true`
- `dump [file]` -> `pg_dump` to file (default: timestamped name)
- `restore <file>` -> **CONFIRM FIRST** -> DROP + CREATE + restore from file
- `query <SQL>` -> run one-off query

---

## Production Database

Access via SSH tunnel through VPS. Auto-discover infrastructure or read `.tunnel-config.json`.

1. Ensure tunnel on port 5433: `gcloud compute ssh <VPS> --zone=<ZONE> -- -L 5433:<SQL_IP>:5432 -N -f`
2. Fetch password: `gcloud secrets versions access latest --secret=db-password`
3. Connect: `PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting`

Operations: `schema`, `tables`, `query <SQL>` (read-only), `dump <table>` (CSV export).

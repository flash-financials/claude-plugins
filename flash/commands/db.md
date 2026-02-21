---
description: Production Cloud SQL access via SSH tunnel
allowed-tools: Bash(gcloud *), Bash(PGPASSWORD=*), Bash(psql *), Bash(lsof *), Bash(sleep *)
argument-hint: "[shell|schema|tables|query <SQL>|dump <table>]"
---

Access the production Cloud SQL database via SSH tunnel through the VPS.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`
- Tunnel status (port 5433): !`lsof -i :5433 -sTCP:LISTEN 2>/dev/null || echo "no tunnel"`
- Tunnel config: !`cat .tunnel-config.json 2>/dev/null || echo "no config"`

## Behavior

Arguments: $ARGUMENTS

**Pre-flight checks:**
- If Active account shows "NOT AUTHENTICATED", stop and tell the user to run `/flash:auth fix` first.
- If GCP project shows "NOT SET", stop and tell the user to run `/flash:auth fix` first.

**Step 1: Discover infrastructure (or read from cache)**

If `.tunnel-config.json` exists in the context, parse it for `sql_ip`, `vps_name`, and `vps_zone`.

Otherwise, discover them:

```bash
# Find Cloud SQL instance private IP
SQL_INSTANCE=$(gcloud sql instances list --format="value(name)" --limit=1)
SQL_IP=$(gcloud sql instances describe $SQL_INSTANCE --format="value(ipAddresses[0].ipAddress)")

# Find VPS name and zone
VPS_INFO=$(gcloud compute instances list --format="value(name,zone)" --filter="name~vps")
VPS_NAME=$(echo "$VPS_INFO" | awk '{print $1}')
VPS_ZONE=$(echo "$VPS_INFO" | awk '{print $2}')
```

**Step 2: Ensure SSH tunnel is open**

Check the tunnel status from context. If "no tunnel":

```bash
gcloud compute ssh $VPS_NAME --zone=$VPS_ZONE -- -L 5433:$SQL_IP:5432 -N -f
sleep 2
```

Verify the tunnel is up:
```bash
lsof -i :5433 -sTCP:LISTEN
```

**Step 3: Fetch DB password**

```bash
DB_PASSWORD=$(gcloud secrets versions access latest --secret=db-password)
```

**Step 4: Execute subcommand**

Connection string: `PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting`

Parse `$ARGUMENTS`:

- **(none)** or **`shell`** → open interactive psql session:
  ```bash
  PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting
  ```

- **`schema`** → show all tables and their columns:
  ```bash
  PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting -c "\dt+"
  ```
  Then for each table:
  ```bash
  PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting -c "\d+ <table>"
  ```

- **`tables`** → list tables with row counts:
  ```bash
  PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting -c "SELECT schemaname, tablename, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC;"
  ```

- **`query <SQL>`** → run a read-only query:
  ```bash
  PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting -c "SET default_transaction_read_only=on; <SQL>"
  ```

- **`dump <table>`** → export table to CSV:
  ```bash
  PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting -c "\copy <table> TO '/tmp/<table>_dump.csv' CSV HEADER"
  ```
  Report the output file path.

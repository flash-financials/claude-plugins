---
name: foundation
description: Connect to FoundationSoft SQL Server, manage SSH tunnels, explore schema, and run queries. Use when the user mentions FoundationSoft, Foundation database, SQL Server, foundation-integration, or job costing data.
---

# FoundationSoft SQL Server

## Context

- Tunnel on port 19000: !`lsof -i :19000 -sTCP:LISTEN 2>/dev/null | tail -1 || echo "NOT ACTIVE"`
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`

## Connection Details

- **Server**: `sql.foundationsoft.com`
- **Port**: `9000`
- **Database**: `Cas_19056`
- **Username**: `LWIN`
- **Password**: `Couts801$`
- **Driver**: SQL Server (go-mssqldb / TDS protocol)
- **Encryption**: disabled

Connection string: `sqlserver://LWIN:Couts801$@127.0.0.1:19000?database=Cas_19056&encrypt=disable`

## SSH Tunnel

FoundationSoft SQL Server is not directly accessible. Route through the GCP VPS:

```bash
gcloud compute ssh flash-financials-vps --project=famous-tree-487406-a7 --zone=us-central1-a -- -L 19000:sql.foundationsoft.com:9000 -N -f
```

Verify: `lsof -i :19000 -sTCP:LISTEN`

Close: `kill $(lsof -t -i :19000 -sTCP:LISTEN)`

Once tunneled, connect via `localhost:19000`.

## GCP Infrastructure

- **Project**: `famous-tree-487406-a7`
- **VPS instance**: `flash-financials-vps`
- **Zone**: `us-central1-a`

---

## Database Schema

### Key Tables

| Table | Purpose |
|-------|---------|
| `jobs` | Job master data (job_no, description, status, dates) |
| `job_budgets` | Original + adjusted budget estimates per job/phase/cost code |
| `job_chg_budgets` | Change order budget adjustments |
| `job_history` | Actual costs and hours posted (labor, material, etc.) |
| `cost_codes` | Cost code master data (descriptions, units of measure) |
| `job_costcodes` | Job-specific cost code assignments |
| `job_phases` | Phase definitions per job |

### Important Columns

**jobs**: `job_no` (char 10), `description`, `job_status`, `job_start_date`, `completion_date`, `customer_no`

**job_budgets**: `job_no`, `phase_no`, `cost_code_no`, `cost_class_no` (char 5), `orig_est_dollars`, `orig_est_units`, `adj_est_dollars`, `adj_est_units`

**job_chg_budgets**: `job_no`, `cost_code_no`, `cost_class_no`, `cost_adj`, `unit_adj`, `change_order_no`

**job_history**: `job_no`, `cost_code_no`, `cost_class_no`, `cost`, `units`, `date_booked`, `date_posted`, `employee_no`, `vendor_no`, `description_1`

**cost_codes**: `cost_code_no`, `description`, `cost_class_no`, `unit_of_measure`

### Cost Classes

Cost classes are stored in `cost_class_no` (char 5, space-padded):
- `'    1'` = **Labor**
- `'    2'` = Material
- `'    3'` = Subcontract
- `'    4'` = Equipment
- `'    5'` through `'    7'` = Other

**All char columns are space-padded.** Always use `LTRIM(RTRIM(...))` when reading string values.

---

## Sample Queries

### List all jobs
```sql
SELECT LTRIM(RTRIM(job_no)) AS job, LTRIM(RTRIM(description)) AS name, job_status
FROM jobs
ORDER BY job_no
```

### Labor budget by job (original + adjustments + change orders)
```sql
SELECT LTRIM(RTRIM(job_no)) AS job,
       ISNULL(SUM(dollars), 0) AS budget_dollars,
       ISNULL(SUM(units), 0) AS budget_hours
FROM (
    SELECT job_no,
           ISNULL(orig_est_dollars, 0) + ISNULL(adj_est_dollars, 0) AS dollars,
           ISNULL(orig_est_units, 0) + ISNULL(adj_est_units, 0) AS units
    FROM job_budgets WHERE cost_class_no = '    1'
    UNION ALL
    SELECT job_no, ISNULL(cost_adj, 0), ISNULL(unit_adj, 0)
    FROM job_chg_budgets WHERE cost_class_no = '    1'
) AS combined
GROUP BY LTRIM(RTRIM(job_no))
ORDER BY LTRIM(RTRIM(job_no))
```

### Actual labor costs by job
```sql
SELECT LTRIM(RTRIM(job_no)) AS job,
       ISNULL(SUM(cost), 0) AS actual_dollars,
       ISNULL(SUM(units), 0) AS actual_hours
FROM job_history
WHERE cost_class_no = '    1'
GROUP BY LTRIM(RTRIM(job_no))
ORDER BY LTRIM(RTRIM(job_no))
```

### Labor detail by cost code for a specific job
```sql
SELECT LTRIM(RTRIM(h.cost_code_no)) AS cost_code,
       LTRIM(RTRIM(c.description)) AS description,
       ISNULL(SUM(h.cost), 0) AS actual_dollars,
       ISNULL(SUM(h.units), 0) AS actual_hours
FROM job_history h
LEFT JOIN cost_codes c ON h.cost_code_no = c.cost_code_no
WHERE h.cost_class_no = '    1'
  AND LTRIM(RTRIM(h.job_no)) = '23048'
GROUP BY h.cost_code_no, c.description
ORDER BY h.cost_code_no
```

### Discover table columns
```sql
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'job_budgets'
ORDER BY ORDINAL_POSITION
```

---

## Operations

### `connect` — Open tunnel and test connection
1. Check if tunnel is already active on port 19000
2. If not, open tunnel via GCP VPS
3. Test connection with a simple query

### `discover <table>` — Show table schema
Run INFORMATION_SCHEMA query for the specified table.

### `query <SQL>` — Run a SQL query
Execute against FoundationSoft via the tunnel. Always use `LTRIM(RTRIM(...))` on char columns.

### `jobs` — List all jobs with status
Quick query of the jobs table.

### `tunnel` — Manage SSH tunnel
- `tunnel open` — Open the SSH tunnel
- `tunnel close` — Close the SSH tunnel
- `tunnel status` — Check if tunnel is active

## Environment Variables (for CLI tools)

```
DB_SERVER=127.0.0.1
DB_PORT=19000
DB_NAME=Cas_19056
DB_USER=LWIN
DB_PASSWORD=Couts801$
```

## Related Projects

- **foundation-integration** (`/Users/scottwinkler/Desktop/feb-24/foundation-integration/`) — Go service that syncs FoundationSoft data to Cloud SQL PostgreSQL
- **flash-excel-reports** — Go CLI that generates Excel labor reports from FoundationSoft data

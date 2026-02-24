---
name: flash
description: Flash Financials dev workflow — local dev, testing, reports, production, CI/CD, secrets, database, infrastructure, and utilities. Use when the user asks about any Flash Financials development, deployment, or operational task.
---

# Flash Financials Development Workflow

## Context

- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`
- Active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`
- Server running: !`lsof -ti :8080 >/dev/null 2>&1 && echo "yes (pid $(lsof -ti :8080 | head -1))" || echo "no"`
- PostgreSQL: !`docker inspect -f '{{.State.Status}}' accounting-db 2>/dev/null || echo "not running"`
- Terraform dir: !`test -d deploy/terraform && echo "exists" || echo "missing"`

If GCP project/region shows "NOT SET" or account shows "NOT AUTHENTICATED", suggest `/flash:auth fix` before proceeding with GCP operations.

## Environment Constants

**Local dev:**
- DB port: `6432`, user: `postgres`, password: `123456`, database: `accounting`
- Server: port `8080`, logs at `/tmp/accounting-server.log`
- Container: `accounting-db`

**GCP (from context above):**
- Registry: `<REGION>-docker.pkg.dev/<PROJECT>/accounting`
- Cloud Run service: `accounting-backend`
- Terraform state bucket: `infra-admin-tfstate`

**Credential files:** `.env`, `token.json`, `client_secret_*.json`
**GSheet credentials:** `client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json`
**GSheet folder:** `1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk`
**Chrome profile:** `Work`
**Vite Google Client ID:** `122638021932-7sd6o9k445tso3vo75vis3i9dh263hba.apps.googleusercontent.com`

---

## Local Development

### Start dev environment

Start Docker + PostgreSQL + server, verify health:

```bash
colima status >/dev/null 2>&1 || colima start
docker-compose up -d
lsof -ti :8080 | xargs kill -9 2>/dev/null || true
DATABASE_URL="host=localhost port=6432 user=postgres password=123456 dbname=accounting sslmode=disable" \
LISTEN_ADDR=":8080" STATIC_DIR="frontend/dist" \
GSHEET_CREDENTIALS="client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
GSHEET_FOLDER="1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
CHROME_PROFILE="Work" SEED_ON_EMPTY=true \
go run ./cmd/server > /tmp/accounting-server.log 2>&1 &
```

Wait 3 seconds, then verify: PostgreSQL (`docker inspect`), server (`lsof -ti :8080`), health (`curl -sf http://localhost:8080/healthz`).

### Stop server

```bash
lsof -ti :8080 | xargs kill -9 2>/dev/null && echo "Stopped" || echo "Not running"
```

Do NOT stop Docker/PostgreSQL unless explicitly asked.

### Restart

Kill server → ensure Docker + DB running → start server fresh → verify. Same commands as start, preceded by the kill.

### Check status

Run three checks: PostgreSQL (`docker inspect`), server (`lsof -ti :8080`), health (`curl -sf http://localhost:8080/healthz`). Report clearly.

### View logs

```bash
tail -100 /tmp/accounting-server.log
```

### Build

```bash
cd frontend && npm ci && npm run build   # Frontend
CGO_ENABLED=0 go build -o bin/accounting ./cmd/reports  # Backend
```

### Format & lint

**Go:** `gofmt -w .` + `go vet ./...` + `golangci-lint run` (if installed)
**Frontend:** `cd frontend && npx prettier --write "src/**/*.{ts,tsx,css,json}"` + `npx tsc --noEmit`
Use `check` argument for dry-run mode (report without fixing).

### Preflight check

Validate the full environment: CLI tools (go >= 1.24, node >= 18, npm, docker, colima, gcloud, gh, psql, terraform), credential files (.env, token.json, client_secret*.json), Docker/colima status, gcloud auth, git state. Present as a checklist with [OK]/[WARN]/[MISS]/[FAIL] markers.

---

## Testing

```bash
go test ./...    # Unit tests
go vet ./...     # Lint
```

**Integration tests** (require PostgreSQL running):
```bash
go test -tags integration -v ./internal/api/... ./internal/repository/postgres/...
```

**Single package shortcuts:** `balance` → `./internal/reports/balance/...`, `income` → `./internal/reports/income/...`, `wip` → `./internal/reports/wip/...`, `forecast` → `./internal/reports/forecast/...`, `api` → `./internal/api/...`, `repo`/`postgres` → `./internal/repository/postgres/...`, `gsheet` → `./internal/integrations/gsheet/...`, `premier` → `./internal/integrations/datasource/premier/...`, `accubuild` → `./internal/integrations/datasource/accubuild/...`, `auth` → `./internal/auth/...`

---

## Reports

### Generate

```bash
go run ./cmd/reports generate \
  --reports <reports> \
  --gsheet-credentials "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
  --gsheet-folder "1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
  --chrome-profile "Work" \
  -y <year> -m <month>
```

Report types: `all` → `wip,forecast,balance,income` | `wip` → `wip,forecast` | `balance` | `income`
Defaults: year = 2025, month = 11.

### Upload to GDrive

```bash
go run ./cmd/reports upload "<file>" \
  --category <category> --type "<type>" \
  --gsheet-credentials "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
  --gsheet-folder "1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
  --chrome-profile "Work"
```

Categories: `raw` (types: wip, financials) | `gold` (types: wip, financials) | `template` (types: wip, forecast, balance, income)

---

## Production

### Deploy to Cloud Run

**IMPORTANT: Always confirm with the user before deploying.**

1. Ensure Docker: `colima status >/dev/null 2>&1 || colima start`
2. Build: `docker build -f deploy/docker/backend.Dockerfile --build-arg VITE_GOOGLE_CLIENT_ID=122638021932-7sd6o9k445tso3vo75vis3i9dh263hba.apps.googleusercontent.com -t <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest .`
3. Push: `docker push <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest`
4. Deploy: `gcloud run deploy accounting-backend --region <REGION> --image <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest`

### Production status

```bash
gcloud run services describe accounting-backend --region <REGION>
```
Extract URL, then health check: `curl -sf <url>/healthz`

### Production logs

```bash
gcloud logging read \
  'resource.type="cloud_run_revision" AND resource.labels.service_name="accounting-backend" AND severity>=<SEVERITY>' \
  --project <PROJECT> --limit <N> --freshness <TIME_RANGE> \
  --format "table(timestamp,severity,textPayload)"
```

Defaults: all severities, 1h, 100 lines. Parse arguments for: `error`/`warning` (severity), `1h`/`6h`/`24h`/`7d` (range), bare number (limit).

Fallback: `gcloud run services logs read accounting-backend --region <REGION> --limit=<N>`

### Execute requests against prod

```bash
SERVICE_URL=$(gcloud run services describe accounting-backend --region <REGION> --format "value(status.url)")
# For /api/* paths, add auth:
TOKEN=$(gcloud auth print-identity-token)
curl -sf -X <METHOD> -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" "$SERVICE_URL<PATH>" [-d '<BODY>']
# For other paths (e.g. /healthz), no auth needed
```

### Rollback

1. List revisions: `gcloud run revisions list --service accounting-backend --region <REGION> --limit 10`
2. Show traffic: `gcloud run services describe accounting-backend --region <REGION> --format "yaml(status.traffic)"`
3. **Confirm with user** which revision to roll back to
4. Shift traffic: `gcloud run services update-traffic accounting-backend --region <REGION> --to-revisions=<REVISION>=100`
5. Verify health

### GCP dashboard

Run comprehensive checks: Cloud Run service status, recent revisions (last 5), health check, Artifact Registry images (last 5), Secret Manager summary. Present as a unified dashboard.

### Cleanup

Clean old Artifact Registry images and Cloud Run revisions. Keep latest 5 of each. **Confirm with user before deleting.** For local Docker: `docker image prune -f`.

---

## CI/CD

### CI status

```bash
gh run list --limit 10
```

### CI logs

Find failed run: `gh run list --status failure --limit 1 --json databaseId,headBranch,name,conclusion,createdAt`

Get errors: `gh run view <id> --log-failed 2>&1 | grep -E "(\[error\]|FAIL|panic:|Error:|cannot |undefined:|syntax error)" | head -30`

Context if needed: `gh run view <id> --log-failed 2>&1 | grep -B5 -A5 "\[error\]" | head -60`

### CI retry

Re-run failed jobs: `gh run rerun <RUN_ID> --failed`

Trigger deploy: `gh workflow run deploy.yml --ref master` (confirm with user first).

---

## Secrets & Auth

### Auth dashboard

Check: account (`gcloud auth list`), project (`gcloud config get-value project`), ADC (`gcloud auth application-default print-access-token`), Docker registry (`~/.docker/config.json` for `docker.pkg.dev`).

**Fixes:** account → `gcloud auth login` | project → `gcloud config set project <ID>` | ADC → `gcloud auth application-default login` | Docker → `gcloud auth configure-docker <REGION>-docker.pkg.dev`

### Check secrets

Required secrets: `jwt-secret`, `google-client-id`, `google-client-secret`, `admin-email`, `accubuild-dsn`, `gsheet-credentials`, `gsheet-token`, `db-password`. Check each with `gcloud secrets versions list <NAME> --project <PROJECT> --limit 1`.

### Push secrets (local → GCP)

Read `.env` and construct AccuBuild DSN from individual vars. Map:
| Local | Secret |
|---|---|
| `$JWT_SECRET` | `jwt-secret` |
| `$GOOGLE_CLIENT_ID` | `google-client-id` |
| `$GOOGLE_CLIENT_SECRET` | `google-client-secret` |
| `$ADMIN_EMAIL` | `admin-email` |
| Constructed DSN | `accubuild-dsn` |
| `client_secret.json` | `gsheet-credentials` |
| `token.json` | `gsheet-token` |

Pattern: `gcloud secrets create <NAME> --project=<PROJECT> 2>/dev/null || true` then `printf '%s' "$VALUE" | gcloud secrets versions add <NAME> --project=<PROJECT> --data-file=-`

### Pull secrets (GCP → local)

Fetch from Secret Manager, parse AccuBuild DSN into individual vars, write `.env`, `token.json`, `client_secret.json`.

---

## Database

### Local database

Connection: `PGPASSWORD=123456 psql -h localhost -p 6432 -U postgres -d accounting`

Operations:
- `schema` → `\dt+` then `\d+ <table>` for each
- `tables` → `SELECT schemaname, tablename, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC`
- `reset` → **CONFIRM FIRST** → DROP + CREATE database, kill server, restart with `/flash:dev`
- `seed` → kill server, restart with `SEED_ON_EMPTY=true`
- `dump [file]` → `pg_dump` to file (default: timestamped name)
- `restore <file>` → **CONFIRM FIRST** → DROP + CREATE + restore from file
- `query <SQL>` → run one-off query

### Production database

Access via SSH tunnel through VPS. Auto-discover infrastructure or read `.tunnel-config.json`.

1. Ensure tunnel on port 5433: `gcloud compute ssh <VPS> --zone=<ZONE> -- -L 5433:<SQL_IP>:5432 -N -f`
2. Fetch password: `gcloud secrets versions access latest --secret=db-password`
3. Connect: `PGPASSWORD=$DB_PASSWORD psql -h localhost -p 5433 -U postgres -d accounting`

Operations: `schema`, `tables`, `query <SQL>` (read-only), `dump <table>` (CSV export).

---

## Infrastructure (Terraform)

All Terraform operations run from `deploy/terraform`. Auto-initialize if `.terraform` doesn't exist.

### Operations

- **status** → `terraform show -no-color`
- **plan** → `terraform plan -no-color`
- **apply** → **CONFIRM FIRST** → plan, then `terraform apply -auto-approve -no-color`
- **output** → `terraform output -no-color`
- **state list** → `terraform state list`
- **state show** → `terraform state show <resource>`
- **drift** → `terraform plan -detailed-exitcode -no-color` (exit 0 = no drift, 2 = drift)

### Initialize (tf-init)

Requires config at `~/.config/terraform-skill/config.json` (created by tf-bootstrap). Adds `backend "gcs"` block to `terraform {}` if missing, then `terraform init` (or `terraform init -migrate-state` if local state exists).

### Bootstrap (tf-bootstrap)

**One-time setup.** Creates infra-admin GCP project + state bucket. Idempotent. Steps: get billing account → create project `infra-admin-<random>` → link billing → enable storage API → create bucket `infra-admin-tfstate` with versioning → save config to `~/.config/terraform-skill/config.json`.

### Scaffold (tf-scaffold)

Create new Terraform project with: `providers.tf` (GCS backend pre-configured), `variables.tf`, `main.tf`, `outputs.tf`, `terraform.tfvars.example`, `.gitignore`. Then `terraform init`.

### State operations (tf-state)

- `list` → `terraform state list`
- `mv <src> <dst>` → `terraform state mv` (confirm first)
- `import <addr> <id>` → `terraform import`
- `rm <addr>` → `terraform state rm` (warn: only removes from state, not the resource)
- `migrate` → add GCS backend block + `terraform init -migrate-state`

### Destroy

**CONFIRM FIRST with strong warning.** `terraform plan -destroy` → show what dies → `terraform destroy -auto-approve`.

---

## Utilities

### Generate dev JWT

```bash
go run ./scripts/gen-token.go
```

Local dev only — hardcoded dev secret, won't work in production.

### Sync plugins to GitHub

Sync local flash plugin to `flash-financials/claude-plugins` repo:

1. Clone: `git clone https://github.com/flash-financials/claude-plugins.git /tmp/claude-plugins-sync`
2. Replace: `rm -rf /tmp/claude-plugins-sync/flash/ && cp -R ~/.claude/plugins/cache/local/flash/1.0.0/ /tmp/claude-plugins-sync/flash/`
3. Update marketplace: `cp ~/.claude/plugins/marketplaces/local/.claude-plugin/marketplace.json /tmp/claude-plugins-sync/marketplace.json`
4. Remove old dirs: `rm -rf /tmp/claude-plugins-sync/{ssh-tunnel,skill-authoring,git-workflow,terraform}/`
5. Show diff, **confirm with user**
6. Commit + push to `main`
7. Clean up temp dir

---
name: flash
description: Flash Financials local development — start/stop/restart the dev server, run tests, build, format, lint, preflight checks, and generate dev JWT tokens.
---

# Flash Financials Local Development

## Context

- Server running: !`lsof -ti :8080 >/dev/null 2>&1 && echo "yes (pid $(lsof -ti :8080 | head -1))" || echo "no"`
- PostgreSQL: !`docker inspect -f '{{.State.Status}}' accounting-db 2>/dev/null || echo "not running"`

## Environment Constants

**Local dev:**
- DB port: `6432`, user: `postgres`, password: `123456`, database: `accounting`
- Server: port `8080`, logs at `/tmp/accounting-server.log`
- Container: `accounting-db`

**Credential files:** `.env`, `token.json`, `client_secret_*.json`
**GSheet credentials:** `client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json`
**GSheet folder:** `1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk`
**Chrome profile:** `Work`

---

## First-time Setup

If credential files are missing (`.env`, `token.json`, `client_secret_*.json`), pull secrets first (see `flash-secrets` skill), then start the dev environment.

---

## Start Dev Environment

Start Docker + PostgreSQL + server, verify health:

```bash
colima status >/dev/null 2>&1 || colima start
docker-compose up -d
lsof -ti :8080 | xargs kill -9 2>/dev/null || true
# Source .env if present (auth vars from secrets-get)
set -a; [ -f .env ] && source .env; set +a
DATABASE_URL="host=localhost port=6432 user=postgres password=123456 dbname=accounting sslmode=disable" \
LISTEN_ADDR=":8080" STATIC_DIR="frontend/dist" \
GSHEET_CREDENTIALS="client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
GSHEET_FOLDER="1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
CHROME_PROFILE="Work" SEED_ON_EMPTY=true \
JWT_SECRET="${JWT_SECRET}" \
GOOGLE_CLIENT_ID="${GOOGLE_CLIENT_ID}" \
GOOGLE_CLIENT_SECRET="${GOOGLE_CLIENT_SECRET}" \
ADMIN_EMAIL="${ADMIN_EMAIL}" \
go run ./cmd/server > /tmp/accounting-server.log 2>&1 &
```

Wait 3 seconds, then verify: PostgreSQL (`docker inspect`), server (`lsof -ti :8080`), health (`curl -sf http://localhost:8080/healthz`).

---

## Stop Server

```bash
lsof -ti :8080 | xargs kill -9 2>/dev/null && echo "Stopped" || echo "Not running"
```

Do NOT stop Docker/PostgreSQL unless explicitly asked.

---

## Restart

Kill server -> ensure Docker + DB running -> source `.env` if present (`set -a; [ -f .env ] && source .env; set +a`) -> start server fresh (with all env vars including auth: `JWT_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `ADMIN_EMAIL`) -> verify.

---

## Check Status

Run three checks: PostgreSQL (`docker inspect`), server (`lsof -ti :8080`), health (`curl -sf http://localhost:8080/healthz`). Report clearly.

---

## View Logs

```bash
tail -100 /tmp/accounting-server.log
```

---

## Build

```bash
cd frontend && npm ci && npm run build   # Frontend
CGO_ENABLED=0 go build -o bin/accounting ./cmd/reports  # Backend
```

---

## Format & Lint

**Go:** `gofmt -w .` + `go vet ./...` + `golangci-lint run` (if installed)
**Frontend:** `cd frontend && npx prettier --write "src/**/*.{ts,tsx,css,json}"` + `npx tsc --noEmit`
Use `check` argument for dry-run mode (report without fixing).

---

## Preflight Check

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

**Single package shortcuts:** `balance` -> `./internal/reports/balance/...`, `income` -> `./internal/reports/income/...`, `wip` -> `./internal/reports/wip/...`, `forecast` -> `./internal/reports/forecast/...`, `api` -> `./internal/api/...`, `repo`/`postgres` -> `./internal/repository/postgres/...`, `gsheet` -> `./internal/integrations/gsheet/...`, `premier` -> `./internal/integrations/datasource/premier/...`, `accubuild` -> `./internal/integrations/datasource/accubuild/...`, `auth` -> `./internal/auth/...`

---

## Generate Dev JWT

```bash
go run ./scripts/gen-token.go
```

Local dev only — hardcoded dev secret, won't work in production.

---
name: test
description: Run tests (unit, package, or integration)
argument-hint: "[package|integration]"
allowed-tools: Bash(go test *), Bash(go vet *)
---

Run tests. Behavior depends on arguments.

Arguments: $ARGUMENTS

**No arguments** — run the full pre-commit suite:
```bash
go test ./...
go vet ./...
```

**`integration`** — run integration tests (requires PostgreSQL to be running):
```bash
go test -tags integration -v ./internal/api/... ./internal/repository/postgres/...
```
If tests fail with connection errors, tell the user to start the DB with `/flash:dev` first. Do NOT auto-start the DB.

**Package name** — run tests for a single package. Common shortcuts:
- `balance` → `go test -v ./internal/reports/balance/...`
- `income` → `go test -v ./internal/reports/income/...`
- `wip` → `go test -v ./internal/reports/wip/...`
- `forecast` → `go test -v ./internal/reports/forecast/...`
- `api` → `go test -v ./internal/api/...`
- `repo` or `postgres` → `go test -v ./internal/repository/postgres/...`
- `gsheet` → `go test -v ./internal/integrations/gsheet/...`
- `premier` → `go test -v ./internal/integrations/datasource/premier/...`
- `accubuild` → `go test -v ./internal/integrations/datasource/accubuild/...`
- `mapping` → `go test -v ./internal/mapping/...`
- `auth` → `go test -v ./internal/auth/...`

If the argument contains `/`, treat it as a direct path: `go test -v ./$ARGUMENTS/...`

If no argument is recognized, ask the user which package they want to test.

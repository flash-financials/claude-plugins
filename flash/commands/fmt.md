---
description: Format and lint Go + frontend code
allowed-tools: Bash(gofmt *), Bash(go vet *), Bash(golangci-lint *), Bash(cd frontend *), Bash(npx *), Bash(which *)
argument-hint: "[go|frontend|ts|check]"
---

Format and lint code across the project.

## Behavior

Arguments: $ARGUMENTS

Parse `$ARGUMENTS`:

- **(none)** → format + lint everything (Go and frontend)
- **`go`** → Go only
- **`frontend`** or **`ts`** → frontend only
- **`check`** → dry-run mode (report issues without fixing)

### Go formatting & linting

**Format** (skip if `check` mode):
```bash
gofmt -w .
```

**In `check` mode**, report only:
```bash
gofmt -l .
```

**Vet:**
```bash
go vet ./...
```

**Lint** (if golangci-lint is installed):
```bash
which golangci-lint && golangci-lint run || echo "golangci-lint not installed, skipping (install with: brew install golangci-lint)"
```

### Frontend formatting & linting

**Format** (skip if `check` mode):
```bash
cd frontend && npx prettier --write "src/**/*.{ts,tsx,css,json}"
```

**In `check` mode**, report only:
```bash
cd frontend && npx prettier --check "src/**/*.{ts,tsx,css,json}"
```

**Type check:**
```bash
cd frontend && npx tsc --noEmit
```

### Report

Summarize results:
```
Format & Lint Summary
======================
Go:
  gofmt:          3 files formatted (or "all clean")
  go vet:         passed (or list issues)
  golangci-lint:  passed (or list issues / skipped)

Frontend:
  prettier:       12 files formatted (or "all clean")
  tsc:            passed (or list errors)
```

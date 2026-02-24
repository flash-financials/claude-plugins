---
name: preflight
description: Validate local environment — tools, credentials, services
allowed-tools: Bash(which *), Bash(go version), Bash(node *), Bash(npm *), Bash(docker *), Bash(colima *), Bash(gcloud *), Bash(gh *), Bash(psql *), Bash(terraform *), Bash(git *), Bash(ls *), Bash(test *)
---

Validate that the local development environment has all required tools, credentials, and services.

## Behavior

Run all checks and present a consolidated checklist.

**1. CLI Tools**

Check each tool exists and get its version:

| Tool | Check | Minimum Version |
|------|-------|-----------------|
| `go` | `which go && go version` | >= 1.24 |
| `node` | `which node && node --version` | >= 18 |
| `npm` | `which npm && npm --version` | any |
| `docker` | `which docker && docker --version` | any |
| `colima` | `which colima && colima version` | any |
| `gcloud` | `which gcloud && gcloud --version 2>/dev/null \| head -1` | any |
| `gh` | `which gh && gh --version` | any |
| `psql` | `which psql && psql --version` | any |
| `terraform` | `which terraform && terraform version \| head -1` | any |

For Go and Node, check minimum versions. Mark `[WARN]` if below minimum, `[MISS]` if not installed.

**2. Credential Files**

```bash
test -f .env && echo "OK" || echo "MISSING"
test -f token.json && echo "OK" || echo "MISSING"
ls client_secret*.json 2>/dev/null || echo "MISSING"
```

If any are missing, suggest: "Run `/flash:secrets-get` to pull credentials from Secret Manager."

**3. Docker / Colima Status**

```bash
colima status 2>&1
docker info --format '{{.ServerVersion}}' 2>/dev/null || echo "NOT RUNNING"
```

Mark `[OK]` if Docker is running, `[FAIL]` if not. Suggest `colima start` if down.

**4. gcloud Auth Status**

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"
gcloud auth application-default print-access-token 2>&1 | head -1
```

Mark `[OK]` if authenticated, `[FAIL]` if not. Suggest `/flash:auth fix` if broken.

**5. Git Repo State**

```bash
git status --porcelain 2>/dev/null | head -5
git branch --show-current 2>/dev/null
```

Report current branch and whether there are uncommitted changes.

**Present the consolidated dashboard:**

```
Preflight Check
================
CLI Tools:
  [OK]   go 1.24.1
  [OK]   node v20.11.0
  [OK]   npm 10.2.4
  [OK]   docker 24.0.7
  [MISS] colima — install with: brew install colima
  [OK]   gcloud 467.0.0
  [OK]   gh 2.40.0
  [MISS] psql — install with: brew install libpq
  [OK]   terraform 1.7.0

Credentials:
  [OK]   .env
  [MISS] token.json — run /flash:secrets-get
  [OK]   client_secret*.json

Services:
  [OK]   Docker running
  [FAIL] gcloud auth expired — run /flash:auth fix

Git:
  [OK]   Branch: main (clean)
```

Summarize: count of OK, WARN, MISS, FAIL. If everything passes, report "Environment ready!"

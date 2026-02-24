---
name: generate
description: Generate financial reports
argument-hint: "[type] [year] [month]"
allowed-tools: Bash(go run *)
---

Generate financial reports.

Arguments: $ARGUMENTS

Parse arguments to determine report type and optional year/month overrides.

**Report types:**
- (no type or `all`) → `--reports wip,forecast,balance,income`
- `wip` → `--reports wip,forecast`
- `balance` → `--reports balance`
- `income` → `--reports income`

**Command template:**
```bash
go run ./cmd/reports generate \
  --reports <reports> \
  --gsheet-credentials "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
  --gsheet-folder "1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
  --chrome-profile "Work" \
  -y <year> -m <month>
```

**Defaults:** year = 2025, month = 11 (unless overridden by arguments).

**Examples:**
- `/flash:generate` → all reports, default year/month
- `/flash:generate wip` → wip + forecast
- `/flash:generate balance 2026 1` → balance sheet for Jan 2026
- `/flash:generate all 2025 12` → all reports for Dec 2025

If the command fails with credential/auth errors, inform the user that Google Drive credentials need to be configured.

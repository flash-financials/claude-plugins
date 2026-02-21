---
description: Upload a file to Google Drive (raw, gold, or template)
argument-hint: "<category> <type> <file>"
allowed-tools: Bash(go run *), AskUserQuestion
---

Upload a file to Google Drive (versioned, converted to Google Sheets).

Arguments: $ARGUMENTS

Format: `/flash:upload <category> <type> <file>`

**Categories and valid types:**
- `raw` — Raw input data. Types: `wip`, `financials`
- `gold` — Gold reference files. Types: `wip`, `financials`
- `template` — Report templates. Types: `wip`, `forecast`, `balance`, `income`

**Command:**
```bash
go run ./cmd/reports upload "<file>" \
  --category <category> --type "<type>" \
  --gsheet-credentials "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
  --gsheet-folder "1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
  --chrome-profile "Work"
```

**Examples:**
- `/flash:upload raw wip ./data/raw/wip-data.xlsx`
- `/flash:upload gold financials ./data/gold/financials_gold.xlsx`
- `/flash:upload template balance ./my-template.xlsx`

If arguments are missing or unclear, ask the user.

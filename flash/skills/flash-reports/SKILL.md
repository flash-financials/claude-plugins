---
name: flash-reports
description: Generate and upload Flash Financials accounting reports — WIP, forecast, balance sheet, income statement. Upload to Google Drive.
---

# Flash Financials Reports

## Context

- GSheet credentials: !`test -f "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" && echo "present" || echo "MISSING"`

## Environment Constants

**GSheet credentials:** `client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json`
**GSheet folder:** `1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk`
**Chrome profile:** `Work`

---

## Generate Reports

```bash
go run ./cmd/reports generate \
  --reports <reports> \
  --gsheet-credentials "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
  --gsheet-folder "1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
  --chrome-profile "Work" \
  -y <year> -m <month>
```

Report types: `all` -> `wip,forecast,balance,income` | `wip` -> `wip,forecast` | `balance` | `income`
Defaults: year = 2025, month = 11.

---

## Upload to Google Drive

```bash
go run ./cmd/reports upload "<file>" \
  --category <category> --type "<type>" \
  --gsheet-credentials "client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
  --gsheet-folder "1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
  --chrome-profile "Work"
```

Categories: `raw` (types: wip, financials) | `gold` (types: wip, financials) | `template` (types: wip, forecast, balance, income)

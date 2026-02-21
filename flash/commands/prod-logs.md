---
description: View production Cloud Run logs with filtering
argument-hint: "[error|warning] [1h|6h|24h] [lines]"
allowed-tools: Bash(gcloud *)
---

View production logs from Cloud Run with optional severity and time filtering.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`

If any context value shows "NOT SET", stop and suggest `/flash:auth fix` first.

Arguments: $ARGUMENTS

Parse the arguments (order-independent):
- **Severity**: `error` or `warning` — filters to that level and above. Default: all levels.
- **Time range**: `1h`, `6h`, `24h`, `7d` etc. — how far back to look. Default: `1h`.
- **Line count**: a bare number like `50` or `200` — max entries to return. Default: `100`.

Examples:
- `/flash:prod-logs` → all levels, last 1h, 100 lines
- `/flash:prod-logs error` → errors only, last 1h, 100 lines
- `/flash:prod-logs error 6h 200` → errors, last 6h, 200 lines
- `/flash:prod-logs 50` → all levels, last 1h, 50 lines

Build the `gcloud logging read` command:
```bash
gcloud logging read \
  'resource.type="cloud_run_revision" AND resource.labels.service_name="accounting-backend"
   AND severity>=<SEVERITY>
   AND timestamp>="<TIMESTAMP>"' \
  --project <PROJECT> \
  --limit <N> \
  --format "table(timestamp,severity,textPayload)" \
  --freshness <TIME_RANGE>
```

Where:
- `<PROJECT>` is from the Context section above
- `<SEVERITY>` is `ERROR`, `WARNING`, or `DEFAULT` (all)
- `<N>` is the line count
- `<TIME_RANGE>` is the time range argument

If `gcloud logging read` fails (e.g. permissions), fall back to the basic command:
```bash
gcloud run services logs read accounting-backend --region <REGION> --limit=<N>
```

Summarize what you see — highlight errors, patterns, or anomalies.

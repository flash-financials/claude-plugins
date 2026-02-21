---
description: Execute HTTP requests against the production endpoint
argument-hint: "<method> <path> [body]"
allowed-tools: Bash(gcloud *), Bash(curl *)
---

Execute an HTTP request against the production Cloud Run endpoint.

## Context
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`

If region shows "NOT SET", stop and suggest `/flash:auth fix` first.

Arguments: $ARGUMENTS

Parse arguments as: `<METHOD> <PATH> [BODY]`
- **METHOD**: HTTP method (GET, POST, PUT, DELETE). Default: `GET`
- **PATH**: URL path (e.g. `/healthz`, `/api/v1/runs`). Default: `/healthz`
- **BODY**: Optional JSON body for POST/PUT requests

Examples:
- `/flash:prod-exec` → GET /healthz
- `/flash:prod-exec GET /api/v1/runs` → list runs
- `/flash:prod-exec POST /api/v1/runs {"year":2025,"month":12}`

**Step 1: Get service URL**
```bash
SERVICE_URL=$(gcloud run services describe accounting-backend --region <REGION> --format "value(status.url)")
```

Replace `<REGION>` with the value from the Context section above.

**Step 2: Build and execute curl command**

For `/api/v1/*` paths, add an auth header:
```bash
TOKEN=$(gcloud auth print-identity-token)
curl -sf -X <METHOD> -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" "$SERVICE_URL<PATH>" [-d '<BODY>']
```

For other paths (like `/healthz`), no auth needed:
```bash
curl -sf -X <METHOD> "$SERVICE_URL<PATH>"
```

**Step 3: Display the response**

Format JSON responses for readability. Report the HTTP status code and highlight any errors.

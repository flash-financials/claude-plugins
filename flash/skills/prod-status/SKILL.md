---
name: prod-status
description: Check production Cloud Run service status and health
allowed-tools: Bash(gcloud *), Bash(curl *)
---

Check the production Cloud Run service status.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`

If any context value shows "NOT SET", stop and suggest `/flash:auth fix` first.

**Service:** `accounting-backend`

Steps:

1. Get service details:
   ```bash
   gcloud run services describe accounting-backend --region <REGION>
   ```

2. Extract the service URL from the output, then health check it:
   ```bash
   curl -sf <url>/healthz
   ```

Replace `<REGION>` with the value from the Context section above.

Report the service status, URL, current image, and health check result.

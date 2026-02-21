---
description: Roll back Cloud Run to a previous revision
allowed-tools: Bash(gcloud *), Bash(curl *), AskUserQuestion
---

Roll back the production Cloud Run service to a previous revision.

## Context
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`

If region shows "NOT SET", stop and suggest `/flash:auth fix` first.

**Step 1: List recent revisions**
```bash
gcloud run revisions list --service accounting-backend --region <REGION> --limit 10 --format "table(name,active,created,status.conditions[0].status)"
```

**Step 2: Show current traffic routing**
```bash
gcloud run services describe accounting-backend --region <REGION> --format "yaml(status.traffic)"
```

**Step 3: Ask user to confirm the target revision**

Present the revisions and current traffic split. Ask the user which revision to roll back to using AskUserQuestion. DO NOT proceed without explicit confirmation.

**Step 4: Shift traffic**
```bash
gcloud run services update-traffic accounting-backend --region <REGION> --to-revisions=<REVISION>=100
```

**Step 5: Verify**

Get the service URL and health check:
```bash
SERVICE_URL=$(gcloud run services describe accounting-backend --region <REGION> --format "value(status.url)")
curl -sf "$SERVICE_URL/healthz"
```

Replace `<REGION>` with the value from the Context section above.

Report the result — confirm the rollback succeeded or flag any issues.

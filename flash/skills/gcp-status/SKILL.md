---
name: gcp-status
description: Comprehensive GCP infrastructure dashboard
allowed-tools: Bash(gcloud *), Bash(curl *)
---

Run a comprehensive GCP health check and present a unified dashboard.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`

If any context value shows "NOT SET", stop and suggest `/flash:auth fix` first.

Use `<PROJECT>` and `<REGION>` from context throughout.

Execute these checks in sequence:

**1. Cloud Run Service Status**
```bash
gcloud run services describe accounting-backend --region <REGION> --format "yaml(status.conditions,status.url,status.traffic)"
```

**2. Recent Revisions (last 5)**
```bash
gcloud run revisions list --service accounting-backend --region <REGION> --limit 5 --format "table(name,active,created,status.conditions[0].status)"
```

**3. Health Check**
```bash
SERVICE_URL=$(gcloud run services describe accounting-backend --region <REGION> --format "value(status.url)")
curl -sf --max-time 10 "$SERVICE_URL/healthz" 2>&1 || echo "HEALTH CHECK FAILED"
```

**4. Artifact Registry Recent Images**
```bash
gcloud artifacts docker images list <REGION>-docker.pkg.dev/<PROJECT>/accounting/accounting-backend --sort-by="~UPDATE_TIME" --limit 5 --format "table(version,updateTime,metadata.imageSizeBytes)"
```

**5. Secret Manager Summary**
```bash
gcloud secrets list --project <PROJECT> --format "table(name,createTime)" 2>&1
```

**Present a unified dashboard** with clear section headers. Highlight any problems:
- Service conditions not "True"
- Health check failures
- Missing or stale images
- Traffic not on latest revision

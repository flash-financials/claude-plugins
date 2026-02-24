---
name: flash-production
description: Flash Financials production operations — deploy to Cloud Run, production status and health, production logs, execute requests against prod, rollback, GCP dashboard, cleanup, CI/CD status, CI logs, and CI retry.
---

# Flash Financials Production & CI/CD

## Context

- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`
- Active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`

If GCP project/region shows "NOT SET" or account shows "NOT AUTHENTICATED", suggest fixing auth before proceeding.

## Environment Constants

- Registry: `<REGION>-docker.pkg.dev/<PROJECT>/accounting`
- Cloud Run service: `accounting-backend`
- Vite Google Client ID: `122638021932-7sd6o9k445tso3vo75vis3i9dh263hba.apps.googleusercontent.com`

---

## Deploy to Cloud Run

**IMPORTANT: Always confirm with the user before deploying.**

1. Ensure Docker: `colima status >/dev/null 2>&1 || colima start`
2. Build: `docker build -f deploy/docker/backend.Dockerfile --build-arg VITE_GOOGLE_CLIENT_ID=122638021932-7sd6o9k445tso3vo75vis3i9dh263hba.apps.googleusercontent.com -t <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest .`
3. Push: `docker push <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest`
4. Deploy: `gcloud run deploy accounting-backend --region <REGION> --image <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest`

---

## Production Status

```bash
gcloud run services describe accounting-backend --region <REGION>
```
Extract URL, then health check: `curl -sf <url>/healthz`

---

## Production Logs

```bash
gcloud logging read \
  'resource.type="cloud_run_revision" AND resource.labels.service_name="accounting-backend" AND severity>=<SEVERITY>' \
  --project <PROJECT> --limit <N> --freshness <TIME_RANGE> \
  --format "table(timestamp,severity,textPayload)"
```

Defaults: all severities, 1h, 100 lines. Parse arguments for: `error`/`warning` (severity), `1h`/`6h`/`24h`/`7d` (range), bare number (limit).

Fallback: `gcloud run services logs read accounting-backend --region <REGION> --limit=<N>`

---

## Execute Requests Against Prod

```bash
SERVICE_URL=$(gcloud run services describe accounting-backend --region <REGION> --format "value(status.url)")
# For /api/* paths, add auth:
TOKEN=$(gcloud auth print-identity-token)
curl -sf -X <METHOD> -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" "$SERVICE_URL<PATH>" [-d '<BODY>']
# For other paths (e.g. /healthz), no auth needed
```

---

## Rollback

1. List revisions: `gcloud run revisions list --service accounting-backend --region <REGION> --limit 10`
2. Show traffic: `gcloud run services describe accounting-backend --region <REGION> --format "yaml(status.traffic)"`
3. **Confirm with user** which revision to roll back to
4. Shift traffic: `gcloud run services update-traffic accounting-backend --region <REGION> --to-revisions=<REVISION>=100`
5. Verify health

---

## GCP Dashboard

Run comprehensive checks: Cloud Run service status, recent revisions (last 5), health check, Artifact Registry images (last 5), Secret Manager summary. Present as a unified dashboard.

---

## Cleanup

Clean old Artifact Registry images and Cloud Run revisions. Keep latest 5 of each. **Confirm with user before deleting.** For local Docker: `docker image prune -f`.

---

## CI/CD

### CI Status

```bash
gh run list --limit 10
```

### CI Logs

Find failed run: `gh run list --status failure --limit 1 --json databaseId,headBranch,name,conclusion,createdAt`

Get errors: `gh run view <id> --log-failed 2>&1 | grep -E "(\[error\]|FAIL|panic:|Error:|cannot |undefined:|syntax error)" | head -30`

Context if needed: `gh run view <id> --log-failed 2>&1 | grep -B5 -A5 "\[error\]" | head -60`

### CI Retry

Re-run failed jobs: `gh run rerun <RUN_ID> --failed`

Trigger deploy: `gh workflow run deploy.yml --ref master` (confirm with user first).

---
name: deploy
description: Build and deploy to Google Cloud Run (confirms first)
allowed-tools: Bash(colima *), Bash(docker build *), Bash(docker push *), Bash(gcloud run deploy *), AskUserQuestion
---

Build and deploy the application to Google Cloud Run.

**IMPORTANT:** This is a production deployment. Confirm with the user before proceeding.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`
- Active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`

If any context value shows "NOT SET" or "NOT AUTHENTICATED", stop and suggest `/flash:auth fix` first.

**GCP values (from context above):**
- Registry: `<REGION>-docker.pkg.dev/<PROJECT>/accounting`
- Service: `accounting-backend`

**Steps:**

1. Ask the user to confirm the deployment.

2. Ensure Docker is running:
   ```bash
   colima status >/dev/null 2>&1 || colima start
   ```

3. Build the Docker image:
   ```bash
   docker build -f deploy/docker/backend.Dockerfile \
     --build-arg VITE_GOOGLE_CLIENT_ID=122638021932-7sd6o9k445tso3vo75vis3i9dh263hba.apps.googleusercontent.com \
     -t <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest .
   ```

4. Push to Artifact Registry:
   ```bash
   docker push <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest
   ```

5. Deploy to Cloud Run:
   ```bash
   gcloud run deploy accounting-backend \
     --region <REGION> \
     --image <REGION>-docker.pkg.dev/<PROJECT>/accounting/backend:latest
   ```

Replace `<PROJECT>` and `<REGION>` with the values from the Context section above.

**Prerequisites:**
- Docker/colima must be running
- Must be authenticated to GCP (`gcloud auth` or `/flash:auth fix`)

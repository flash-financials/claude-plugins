---
description: Pull secrets from GCP Secret Manager to local dev files
allowed-tools: Bash(gcloud *), Bash(printf *), Bash(echo *)
---

Pull secrets from GCP Secret Manager and write them to local `.env`, `token.json`, and `client_secret.json`.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`

If project shows "NOT SET", stop and suggest `/flash:auth fix` first.

Use `<PROJECT>` from context throughout.

**Steps:**

1. Fetch all secrets from Secret Manager:
   ```bash
   JWT_SECRET=$(gcloud secrets versions access latest --secret=jwt-secret --project=<PROJECT>)
   GOOGLE_CLIENT_ID=$(gcloud secrets versions access latest --secret=google-client-id --project=<PROJECT>)
   GOOGLE_CLIENT_SECRET=$(gcloud secrets versions access latest --secret=google-client-secret --project=<PROJECT>)
   ADMIN_EMAIL=$(gcloud secrets versions access latest --secret=admin-email --project=<PROJECT>)
   ACCUBUILD_DSN=$(gcloud secrets versions access latest --secret=accubuild-dsn --project=<PROJECT>)
   ```

2. Parse the AccuBuild DSN (`user:pass@tcp(host:port)/db`) into individual vars:
   ```bash
   ACCUBUILD_USER=$(echo "$ACCUBUILD_DSN" | sed -E 's/^([^:]+):.*$/\1/')
   ACCUBUILD_PASSWORD=$(echo "$ACCUBUILD_DSN" | sed -E 's/^[^:]+:(.*)@tcp\(.*/\1/')
   ACCUBUILD_HOST=$(echo "$ACCUBUILD_DSN" | sed -E 's/.*@tcp\(([^)]+)\).*/\1/')
   ACCUBUILD_DB=$(echo "$ACCUBUILD_DSN" | sed -E 's/.*\///')
   ```

3. Write `.env`:
   ```bash
   cat > .env << ENVEOF
   # Auth (Google OAuth2 + JWT)
   JWT_SECRET=${JWT_SECRET}
   GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}
   GOOGLE_CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}
   ADMIN_EMAIL=${ADMIN_EMAIL}

   # Frontend auth
   VITE_GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}

   # AccuBuild MySQL (Google Cloud SQL)
   # Use separate vars to avoid password encoding issues with special chars (/, %, ^)
   ACCUBUILD_USER=${ACCUBUILD_USER}
   ACCUBUILD_PASSWORD=${ACCUBUILD_PASSWORD}
   ACCUBUILD_HOST=${ACCUBUILD_HOST}
   ACCUBUILD_DB=${ACCUBUILD_DB}
   ENVEOF
   ```

4. Write `token.json`:
   ```bash
   gcloud secrets versions access latest --secret=gsheet-token --project=<PROJECT> > token.json
   ```

5. Write `client_secret.json`:
   ```bash
   gcloud secrets versions access latest --secret=gsheet-credentials --project=<PROJECT> > client_secret.json
   ```

6. Confirm success by listing the written files:
   ```bash
   ls -la .env token.json client_secret.json
   ```

**Prerequisites:**
- Must be authenticated to GCP (`gcloud auth login` or `/flash:auth fix`)
- Secrets must already exist in Secret Manager

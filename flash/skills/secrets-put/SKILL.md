---
name: secrets-put
description: Push local secrets to GCP Secret Manager
allowed-tools: Bash(gcloud *), Bash(cat *), Bash(grep *), Bash(printf *), Bash(echo *), AskUserQuestion
---

Read local `.env`, `token.json`, and `client_secret.json`, then push each value to GCP Secret Manager.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`

If project shows "NOT SET", stop and suggest `/flash:auth fix` first.

Use `<PROJECT>` from context throughout.

**Steps:**

1. Read `.env` and show a summary of which keys will be pushed (names only, **not values**). Also check for `token.json` and `client_secret.json`. Ask the user to confirm before proceeding.

2. Build the AccuBuild DSN from individual env vars:
   ```bash
   # Source the .env file
   set -a; source .env; set +a
   ACCUBUILD_DSN="${ACCUBUILD_USER}:${ACCUBUILD_PASSWORD}@tcp(${ACCUBUILD_HOST})/${ACCUBUILD_DB}"
   ```

3. For each secret, create it if it doesn't exist, then add a new version. Use this pattern for each:
   ```bash
   gcloud secrets create SECRET_NAME --project=<PROJECT> 2>/dev/null || true
   printf '%s' "$VALUE" | gcloud secrets versions add SECRET_NAME --project=<PROJECT> --data-file=-
   ```

   Secret mapping:
   | Local Source | Secret Name |
   |---|---|
   | `$JWT_SECRET` | `jwt-secret` |
   | `$GOOGLE_CLIENT_ID` | `google-client-id` |
   | `$GOOGLE_CLIENT_SECRET` | `google-client-secret` |
   | `$ADMIN_EMAIL` | `admin-email` |
   | `$ACCUBUILD_DSN` (constructed) | `accubuild-dsn` |
   | `client_secret.json` (file) | `gsheet-credentials` |
   | `token.json` (file) | `gsheet-token` |

4. For file-based secrets, use `--data-file` directly:
   ```bash
   gcloud secrets versions add gsheet-credentials --project=<PROJECT> --data-file=client_secret.json
   gcloud secrets versions add gsheet-token --project=<PROJECT> --data-file=token.json
   ```

5. Confirm success by listing secret versions:
   ```bash
   for s in jwt-secret google-client-id google-client-secret admin-email accubuild-dsn gsheet-credentials gsheet-token; do
     echo "$s: $(gcloud secrets versions list $s --project=<PROJECT> --limit=1 --format='value(name)' 2>/dev/null || echo 'NOT FOUND')"
   done
   ```

**Prerequisites:**
- `.env`, `token.json`, and `client_secret.json` must exist in the working directory
- Must be authenticated to GCP (`gcloud auth login` or `/flash:auth fix`)

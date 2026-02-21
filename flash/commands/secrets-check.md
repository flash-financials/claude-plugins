---
description: Validate required secrets exist in GCP Secret Manager
allowed-tools: Bash(gcloud *)
---

Check that all required secrets exist in GCP Secret Manager and have active versions.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`

If project shows "NOT SET", stop and suggest `/flash:auth fix` first.

Use `<PROJECT>` from context throughout.

**Required secrets:**
- `jwt-secret`
- `google-client-id`
- `google-client-secret`
- `admin-email`
- `accubuild-dsn`
- `gsheet-credentials`
- `gsheet-token`
- `db-password`

**Step 1: List all secrets**
```bash
gcloud secrets list --project <PROJECT> --format "table(name,createTime)" 2>&1
```

**Step 2: For each required secret, check if it exists and has an active version**
```bash
gcloud secrets versions list <SECRET_NAME> --project <PROJECT> --limit 1 --format "value(name,state)" 2>&1
```

**Step 3: Report results as a checklist**

Format:
```
Secret Health Check
===================
[OK]      jwt-secret (version 3, ENABLED)
[OK]      google-client-id (version 1, ENABLED)
[MISSING] accubuild-dsn — not found
[DISABLED] gsheet-token (version 2, DISABLED)
```

If any secrets are missing or disabled, suggest using `/flash:secrets-put` to fix them.

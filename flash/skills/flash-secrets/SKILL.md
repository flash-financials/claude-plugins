---
name: flash-secrets
description: Flash Financials secrets and authentication — check GCP auth status, fix auth issues, manage secrets in Google Secret Manager, push secrets to GCP, pull secrets to local .env and credential files.
---

# Flash Financials Secrets & Auth

## Context

- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`
- Active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`

## Environment Constants

- Registry: `<REGION>-docker.pkg.dev/<PROJECT>/accounting` (for Docker auth)
- Credential files: `.env`, `token.json`, `client_secret_*.json`
- Required secrets: `jwt-secret`, `encryption-key`, `google-client-id`, `google-client-secret`, `admin-email`, `allowed-domains`, `allowed-emails`, `accubuild-dsn`, `gsheet-credentials`, `gsheet-token`, `db-password`

---

## Auth Dashboard

Check: account (`gcloud auth list`), project (`gcloud config get-value project`), ADC (`gcloud auth application-default print-access-token`), Docker registry (`~/.docker/config.json` for `docker.pkg.dev`).

**Fixes:** account -> `gcloud auth login` | project -> `gcloud config set project <ID>` | ADC -> `gcloud auth application-default login` | Docker -> `gcloud auth configure-docker <REGION>-docker.pkg.dev`

---

## Check Secrets

Required secrets: `jwt-secret`, `encryption-key`, `google-client-id`, `google-client-secret`, `admin-email`, `allowed-domains`, `allowed-emails`, `accubuild-dsn`, `gsheet-credentials`, `gsheet-token`, `db-password`. Check each with `gcloud secrets versions list <NAME> --project <PROJECT> --limit 1`.

---

## Push Secrets (local -> GCP)

Read `.env` and construct AccuBuild DSN from individual vars. Map:
| Local | Secret |
|---|---|
| `$JWT_SECRET` | `jwt-secret` |
| `$ENCRYPTION_KEY` | `encryption-key` |
| `$GOOGLE_CLIENT_ID` | `google-client-id` |
| `$GOOGLE_CLIENT_SECRET` | `google-client-secret` |
| `$ADMIN_EMAIL` | `admin-email` |
| `$ALLOWED_DOMAINS` | `allowed-domains` |
| `$ALLOWED_EMAILS` | `allowed-emails` |
| Constructed DSN | `accubuild-dsn` |
| `client_secret.json` | `gsheet-credentials` |
| `token.json` | `gsheet-token` |

Pattern: `gcloud secrets create <NAME> --project=<PROJECT> 2>/dev/null || true` then `printf '%s' "$VALUE" | gcloud secrets versions add <NAME> --project=<PROJECT> --data-file=-`

---

## Pull Secrets (GCP -> local)

Fetch from Secret Manager, parse AccuBuild DSN into individual vars. Write credential files:
- `.env` <- parsed env vars
- `token.json` <- gsheet-token secret
- `client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json` <- gsheet-credentials secret

---
description: "One-time setup: create infra-admin GCP project and Terraform state bucket"
disable-model-invocation: true
allowed-tools: Bash(gcloud *), Bash(gsutil *), Bash(cat *), Bash(mkdir *), Bash(jq *), Read, Write, AskUserQuestion
---

## Context

- gcloud active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`
- gcloud default project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Existing config: !`cat ~/.config/terraform-skill/config.json 2>/dev/null || echo "No config found"`

## Your Task

Bootstrap the Terraform infrastructure admin project and state bucket. This is idempotent — if already set up, it reports status and exits.

### Step 1: Check Prerequisites

- If gcloud is NOT AUTHENTICATED, tell the user to run `gcloud auth login` and stop.

### Step 2: Check Existing Config

Read the config from context above. If config exists AND contains a valid `project_id` and `bucket`:
1. Verify the project still exists: `gcloud projects describe <project_id> --format="value(projectId)" 2>/dev/null`
2. Verify the bucket still exists: `gcloud storage buckets describe gs://<bucket> --project=<project_id> --format="value(name)" 2>/dev/null`
3. If both exist, report "Already bootstrapped" with the details and stop.
4. If either is missing, continue to recreate.

### Step 3: Get Billing Account

Ask the user for their GCP billing account ID. Suggest they can find it with:
```bash
gcloud billing accounts list --format="table(name,displayName,open)"
```

### Step 4: Create Admin Project

Generate a project ID: `infra-admin-<6-random-digits>` (e.g., `infra-admin-847291`).

```bash
gcloud projects create <PROJECT_ID> --name="Infrastructure Admin" --set-as-project=false
```

If this fails because the project already exists (from a previous partial bootstrap), try to use it.

### Step 5: Link Billing

```bash
gcloud billing projects link <PROJECT_ID> --billing-account=<BILLING_ACCOUNT>
```

### Step 6: Enable Storage API

```bash
gcloud services enable storage.googleapis.com --project=<PROJECT_ID>
```

Wait 10 seconds for API to propagate.

### Step 7: Create State Bucket

```bash
gcloud storage buckets create gs://infra-admin-tfstate \
  --project=<PROJECT_ID> \
  --location=us \
  --uniform-bucket-level-access \
  --public-access-prevention
```

If the bucket name is taken (globally unique), try `infra-admin-tfstate-<6-random-digits>`.

Enable versioning:
```bash
gcloud storage buckets update gs://<BUCKET_NAME> --versioning
```

### Step 8: Save Config

Create `~/.config/terraform-skill/config.json`:

```json
{
  "project_id": "<PROJECT_ID>",
  "bucket": "<BUCKET_NAME>",
  "created_at": "<ISO_TIMESTAMP>",
  "created_by": "<GCLOUD_ACCOUNT>"
}
```

### Step 9: Report

Show the user:
- Admin project ID
- State bucket name
- Config file location
- Next steps: "Run `/terraform:init` in any Terraform project to configure remote state"

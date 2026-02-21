---
description: Initialize Terraform with remote GCS backend
allowed-tools: Bash(terraform *), Bash(cat *), Bash(basename *), Bash(pwd), Read, Write, Edit, AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Directory name: !`basename $(pwd)`
- Terraform skill config: !`cat ~/.config/terraform-skill/config.json 2>/dev/null || echo "No config found"`
- Backend configured: !`grep -l 'backend "gcs"' *.tf 2>/dev/null || echo "No backend block found"`
- Local state exists: !`ls terraform.tfstate 2>/dev/null || echo "No local state"`
- .terraform exists: !`ls -d .terraform 2>/dev/null || echo "Not initialized"`

## Your Task

Initialize Terraform in the current directory with remote GCS backend.

### Step 1: Check Prerequisites

- If config shows "No config found", tell the user to run `/terraform:bootstrap` first and stop.
- If no `.tf` files exist in the current directory, tell the user this doesn't look like a Terraform project and stop.

### Step 2: Check Backend Configuration

Read the config to get the bucket name and the directory name for the state prefix.

If "No backend block found" in context:
1. Find `providers.tf` (or the file containing the `terraform {}` block)
2. Add a backend block inside the `terraform {}` block:

```hcl
  backend "gcs" {
    bucket = "<BUCKET_FROM_CONFIG>"
    prefix = "<DIRECTORY_NAME>"
  }
```

3. Show the user what was added and confirm before proceeding.

### Step 3: Initialize

If local state exists (terraform.tfstate file found):
```bash
terraform init -migrate-state
```
This will prompt to migrate local state to the remote backend. Answer "yes".

If no local state:
```bash
terraform init
```

### Step 4: Verify

```bash
terraform state list
```

Report success and show the remote state location: `gs://<BUCKET>/<PREFIX>/default.tfstate`

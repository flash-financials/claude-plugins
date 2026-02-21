---
description: Destroy Terraform-managed infrastructure (confirms first)
disable-model-invocation: true
allowed-tools: Bash(terraform *), Bash(ls *), AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Initialized: !`ls -d .terraform 2>/dev/null || echo "NOT INITIALIZED"`

## Your Task

Destroy all Terraform-managed infrastructure with explicit confirmation.

### Step 1: Check Prerequisites

- If "NOT INITIALIZED", tell the user to run `/terraform:init` first and stop.

### Step 2: Preview Destruction

```bash
terraform plan -destroy
```

Show the user exactly what will be destroyed.

### Step 3: Confirm

Ask the user to confirm destruction. Use strong warning language — this is irreversible. List all resources that will be destroyed.

### Step 4: Destroy

```bash
terraform destroy -auto-approve
```

### Step 5: Report

Report what was destroyed. Remind the user that the state file in the remote backend still exists (for audit purposes) and can be cleaned up with `/terraform:state rm` if desired.

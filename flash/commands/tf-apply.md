---
description: Apply Terraform changes (confirms first)
disable-model-invocation: true
allowed-tools: Bash(terraform *), Bash(ls *), AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Initialized: !`ls -d .terraform 2>/dev/null || echo "NOT INITIALIZED"`

## Your Task

Apply Terraform changes with user confirmation.

### Step 1: Check Prerequisites

- If "NOT INITIALIZED", tell the user to run `/flash:tf-init` first and stop.

### Step 2: Preview Changes

```bash
terraform plan
```

Show the summary of changes to the user.

### Step 3: Confirm

Ask the user to confirm they want to apply these changes. Warn if any resources will be destroyed.

### Step 4: Apply

```bash
terraform apply -auto-approve
```

### Step 5: Show Outputs

```bash
terraform output
```

Report success and show all outputs.

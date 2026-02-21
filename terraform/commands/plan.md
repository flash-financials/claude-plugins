---
description: Run terraform plan to preview changes
allowed-tools: Bash(terraform *), Bash(ls *)
---

## Context

- Current directory: !`pwd`
- Initialized: !`ls -d .terraform 2>/dev/null || echo "NOT INITIALIZED"`
- Terraform files: !`ls *.tf 2>/dev/null || echo "No .tf files found"`

## Your Task

Run `terraform plan` to preview infrastructure changes.

### Step 1: Check Prerequisites

- If "NOT INITIALIZED", tell the user to run `/terraform:init` first and stop.
- If "No .tf files found", tell the user this doesn't look like a Terraform project and stop.

### Step 2: Run Plan

```bash
terraform plan
```

### Step 3: Summarize

Summarize the planned changes:
- Resources to add
- Resources to change
- Resources to destroy

If no changes, report "Infrastructure is up to date."

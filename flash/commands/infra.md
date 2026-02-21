---
description: Terraform operations for Flash infrastructure
allowed-tools: Bash(terraform *), Bash(ls *), Bash(test *), Bash(cd *), AskUserQuestion
argument-hint: "[status|plan|apply|output|state list|state show <resource>|drift]"
---

Run Terraform operations for the Flash Financials infrastructure.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Terraform dir exists: !`test -d deploy/terraform && echo "yes" || echo "no"`
- Terraform initialized: !`test -d deploy/terraform/.terraform && echo "yes" || echo "no"`

## Behavior

Arguments: $ARGUMENTS

**Pre-flight checks:**
- If GCP project shows "NOT SET", stop and suggest `/flash:auth fix`.
- If Terraform dir does not exist, stop and report: "`deploy/terraform` directory not found."
- Terraform directory: `deploy/terraform`

**Auto-initialize if needed:**

If Terraform is not initialized (context shows "no" for initialized), run:
```bash
cd deploy/terraform && terraform init -backend-config="bucket=<PROJECT>-terraform-state"
```
Where `<PROJECT>` is the GCP project from context.

**All terraform commands run from `deploy/terraform`.**

Parse `$ARGUMENTS`:

- **(none)** or **`status`** → show current state summary:
  ```bash
  cd deploy/terraform && terraform show -no-color
  ```
  Summarize the resources that exist.

- **`plan`** → run terraform plan:
  ```bash
  cd deploy/terraform && terraform plan -no-color
  ```
  Summarize the planned changes (resources to add, change, destroy).

- **`apply`** → **DESTRUCTIVE: Confirm with user first using AskUserQuestion!**
  1. First run plan so user can see what will change:
     ```bash
     cd deploy/terraform && terraform plan -no-color
     ```
  2. Ask: "Apply these Terraform changes? This will modify production infrastructure."
  3. If confirmed:
     ```bash
     cd deploy/terraform && terraform apply -auto-approve -no-color
     ```

- **`output`** → show all outputs:
  ```bash
  cd deploy/terraform && terraform output -no-color
  ```

- **`state list`** → list all resources in state:
  ```bash
  cd deploy/terraform && terraform state list
  ```

- **`state show <resource>`** → show details of a specific resource:
  ```bash
  cd deploy/terraform && terraform state show <resource>
  ```

- **`drift`** → check for infrastructure drift:
  ```bash
  cd deploy/terraform && terraform plan -detailed-exitcode -no-color
  ```
  Exit code 0 = no drift, exit code 2 = drift detected.
  Report whether drift was found and summarize any differences.

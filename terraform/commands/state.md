---
description: "Terraform state operations (list, mv, import, rm, migrate)"
argument-hint: "[list|mv|import|rm|migrate] [args...]"
allowed-tools: Bash(terraform *), Bash(cat *), Bash(basename *), Bash(pwd), Bash(ls *), Read, Write, Edit, AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Directory name: !`basename $(pwd)`
- Terraform skill config: !`cat ~/.config/terraform-skill/config.json 2>/dev/null || echo "No config found"`
- Initialized: !`ls -d .terraform 2>/dev/null || echo "NOT INITIALIZED"`
- Local state exists: !`ls terraform.tfstate 2>/dev/null || echo "No local state"`
- Backend configured: !`grep -l 'backend "gcs"' *.tf 2>/dev/null || echo "No backend block"`

## Your Task

Perform Terraform state operations. Parse the arguments to determine the operation.

### Operations

**`list`** — List all resources in state:
```bash
terraform state list
```

**`mv <source> <destination>`** — Move/rename a resource in state:
```bash
terraform state mv <source> <destination>
```
Confirm with user before executing.

**`import <address> <id>`** — Import an existing resource into state:
```bash
terraform import <address> <id>
```
Show the user the resource address and ID before importing.

**`rm <address>`** — Remove a resource from state (does NOT destroy the actual resource):
```bash
terraform state rm <address>
```
Warn the user that this only removes it from state management, the actual resource remains.

**`migrate`** — Migrate local state to remote GCS backend:

1. If config shows "No config found", tell the user to run `/terraform:bootstrap` first and stop.
2. If "No local state" and "No backend block", nothing to migrate. Stop.
3. If backend is already configured and no local state, already migrated. Stop.
4. Read config to get bucket name.
5. Add `backend "gcs"` block to the terraform block in providers.tf:
   ```hcl
     backend "gcs" {
       bucket = "<BUCKET>"
       prefix = "<DIRECTORY_NAME>"
     }
   ```
6. Run `terraform init -migrate-state` and answer "yes" to migrate.
7. Verify with `terraform state list`.
8. Report success.

### No Arguments

If no arguments provided, ask the user which operation they want to perform.

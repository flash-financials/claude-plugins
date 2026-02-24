---
name: flash-infra
description: Flash Financials Terraform infrastructure — plan, apply, state management, drift detection, initialize, bootstrap, scaffold, and destroy.
---

# Flash Financials Infrastructure (Terraform)

## Context

- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`
- Terraform dir: !`test -d deploy/terraform && echo "exists" || echo "missing"`

## Environment Constants

- Terraform state bucket: `infra-admin-tfstate`
- Working directory: `deploy/terraform`

All Terraform operations run from `deploy/terraform`. Auto-initialize if `.terraform` doesn't exist.

---

## Operations

- **status** -> `terraform show -no-color`
- **plan** -> `terraform plan -no-color`
- **apply** -> **CONFIRM FIRST** -> plan, then `terraform apply -auto-approve -no-color`
- **output** -> `terraform output -no-color`
- **state list** -> `terraform state list`
- **state show** -> `terraform state show <resource>`
- **drift** -> `terraform plan -detailed-exitcode -no-color` (exit 0 = no drift, 2 = drift)

---

## Initialize (tf-init)

Requires config at `~/.config/terraform-skill/config.json` (created by tf-bootstrap). Adds `backend "gcs"` block to `terraform {}` if missing, then `terraform init` (or `terraform init -migrate-state` if local state exists).

---

## Bootstrap (tf-bootstrap)

**One-time setup.** Creates infra-admin GCP project + state bucket. Idempotent. Steps: get billing account -> create project `infra-admin-<random>` -> link billing -> enable storage API -> create bucket `infra-admin-tfstate` with versioning -> save config to `~/.config/terraform-skill/config.json`.

---

## Scaffold (tf-scaffold)

Create new Terraform project with: `providers.tf` (GCS backend pre-configured), `variables.tf`, `main.tf`, `outputs.tf`, `terraform.tfvars.example`, `.gitignore`. Then `terraform init`.

---

## State Operations (tf-state)

- `list` -> `terraform state list`
- `mv <src> <dst>` -> `terraform state mv` (confirm first)
- `import <addr> <id>` -> `terraform import`
- `rm <addr>` -> `terraform state rm` (warn: only removes from state, not the resource)
- `migrate` -> add GCS backend block + `terraform init -migrate-state`

---

## Destroy

**CONFIRM FIRST with strong warning.** `terraform plan -destroy` -> show what dies -> `terraform destroy -auto-approve`.

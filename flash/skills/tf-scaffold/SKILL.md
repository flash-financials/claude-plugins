---
name: tf-scaffold
description: Scaffold a new Terraform project with remote GCS backend
argument-hint: "[project-name]"
allowed-tools: Bash(terraform *), Bash(mkdir *), Bash(ls *), Read, Write, AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Terraform skill config: !`cat ~/.config/terraform-skill/config.json 2>/dev/null || echo "No config found"`

## Your Task

Create a new Terraform project directory with remote GCS backend pre-configured.

### Step 1: Check Prerequisites

- If config shows "No config found", tell the user to run `/flash:tf-bootstrap` first and stop.

### Step 2: Get Project Name

If an argument was provided, use it as the project name. Otherwise, ask the user for a project name (e.g., `gcp-workstation`, `my-app-infra`).

### Step 3: Create Directory Structure

Read the config to get the bucket name. Create the following files:

**`<project-name>/providers.tf`:**
```hcl
terraform {
  required_version = ">= 1.6.0"

  backend "gcs" {
    bucket = "<BUCKET_FROM_CONFIG>"
    prefix = "<PROJECT_NAME>"
  }

  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

**`<project-name>/variables.tf`:**
```hcl
variable "project_id" {
  description = "GCP project ID"
  type        = string
}

variable "region" {
  description = "GCP region"
  type        = string
  default     = "us-central1"
}
```

**`<project-name>/main.tf`:**
```hcl
# Add your resources here
```

**`<project-name>/outputs.tf`:**
```hcl
# Add your outputs here
```

**`<project-name>/terraform.tfvars.example`:**
```hcl
project_id = "my-project-id"
# region   = "us-central1"
```

**`<project-name>/.gitignore`:**
```
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
!*.tfvars.example
.terraform.lock.hcl
```

### Step 4: Initialize

```bash
cd <project-name> && terraform init
```

### Step 5: Report

Show the user:
- Created directory structure
- Remote state location: `gs://<BUCKET>/<PROJECT_NAME>/default.tfstate`
- Next steps: edit `main.tf`, copy `terraform.tfvars.example` to `terraform.tfvars` and fill in values

---
name: cleanup
description: Clean up old Artifact Registry images and Cloud Run revisions
allowed-tools: Bash(gcloud *), Bash(docker *), AskUserQuestion
argument-hint: "[all|images|revisions|local]"
---

Clean up old container images and Cloud Run revisions to reduce costs and clutter.

## Context
- GCP project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- Region: !`gcloud config get-value compute/region 2>/dev/null || echo "NOT SET"`

## Behavior

Arguments: $ARGUMENTS

If GCP project shows "NOT SET", stop and suggest `/flash:auth fix`.

Parse `$ARGUMENTS`:

- **(none)** or **`all`** → clean both images and revisions
- **`images`** → Artifact Registry only
- **`revisions`** → Cloud Run revisions only
- **`local`** → local Docker image prune + disk stats

### Artifact Registry cleanup (`images` or `all`)

**Step 1:** Discover repos:
```bash
gcloud artifacts repositories list --format="value(name)" --project=<PROJECT>
```

**Step 2:** For each repo, list images sorted by time:
```bash
gcloud artifacts docker images list <REGION>-docker.pkg.dev/<PROJECT>/<REPO> --sort-by="~UPDATE_TIME" --format="table(version,updateTime)" --include-tags
```

**Step 3:** Identify images to delete — keep the 5 most recent, mark the rest for deletion.

**Step 4:** Present the list to the user via AskUserQuestion: "Delete N old images from Artifact Registry? (keeping latest 5)"

**Step 5:** If confirmed, delete old images:
```bash
gcloud artifacts docker images delete <IMAGE_PATH>@<DIGEST> --delete-tags --quiet
```

### Cloud Run revision cleanup (`revisions` or `all`)

**Step 1:** Discover services:
```bash
gcloud run services list --format="value(name)" --region=<REGION> --project=<PROJECT>
```

**Step 2:** For each service, list revisions:
```bash
gcloud run revisions list --service=<SERVICE> --region=<REGION> --format="table(name,active,created)" --project=<PROJECT>
```

**Step 3:** Identify non-active revisions to delete — keep the 5 most recent, mark the rest.

**Step 4:** Present to user: "Delete N old revisions? (keeping latest 5 + active)"

**Step 5:** If confirmed:
```bash
gcloud run revisions delete <REVISION_NAME> --region=<REGION> --quiet --project=<PROJECT>
```

### Local Docker cleanup (`local`)

```bash
docker system df
docker image prune -f
docker system df
```

Report space reclaimed.

### Summary

Report what was cleaned up:
```
Cleanup Summary
================
Artifact Registry: deleted 12 images, kept 5
Cloud Run:         deleted 8 revisions, kept 5 + active
Local Docker:      reclaimed 2.3 GB
```

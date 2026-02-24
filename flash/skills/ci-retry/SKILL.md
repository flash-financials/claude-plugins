---
name: ci-retry
description: Retry failed CI runs or trigger deploy workflow
allowed-tools: Bash(gh *), AskUserQuestion
argument-hint: "[<run-id>|deploy]"
---

Retry failed GitHub Actions runs or trigger the deploy workflow.

## Behavior

Arguments: $ARGUMENTS

Parse `$ARGUMENTS`:

- **(none)** → find the most recent failed run and offer to re-run it
- **`<run-id>`** (a number) → re-run that specific run
- **`deploy`** → trigger the deploy workflow manually

### Default: Re-run most recent failed run

**Step 1:** Find the most recent failed workflow run:
```bash
gh run list --status failure --limit 5 --json databaseId,displayTitle,workflowName,createdAt,headBranch
```

**Step 2:** Show the failed runs to the user and ask which one to re-run using AskUserQuestion. Include the run title, workflow name, branch, and date.

**Step 3:** Re-run only the failed jobs:
```bash
gh run rerun <RUN_ID> --failed
```

**Step 4:** Report the re-run status and suggest:
"Watch progress with `/flash:ci-logs <RUN_ID>`"

### Specific run ID

If `$ARGUMENTS` is a number:
```bash
gh run rerun $ARGUMENTS --failed
```

Report success and suggest `/flash:ci-logs $ARGUMENTS` to watch.

### Deploy workflow

If `$ARGUMENTS` is `deploy`:

**Step 1:** Confirm with the user using AskUserQuestion:
"This will trigger the deploy workflow on the master branch. This deploys to production. Continue?"

**Step 2:** If confirmed:
```bash
gh workflow run deploy.yml --ref master
```

**Step 3:** Wait a moment, then find the new run ID:
```bash
gh run list --workflow=deploy.yml --limit 1 --json databaseId,status,createdAt
```

Report the run ID and suggest `/flash:ci-logs <RUN_ID>` to watch progress.

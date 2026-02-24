---
name: ci-logs
description: View logs from a failed GitHub Actions run
argument-hint: "[run-id]"
allowed-tools: Bash(gh *)
---

Get logs from a failed GitHub Actions workflow run.

Arguments: $ARGUMENTS

## Step 1: Find the failed run

**If a run ID is provided**, use it directly.

**If no run ID is provided**, find the most recent failed run:
```bash
gh run list --status failure --limit 1 --json databaseId,headBranch,name,conclusion,createdAt
```

## Step 2: Get the error summary first

Always start with a filtered view to find the actual errors quickly — full logs are extremely verbose:
```bash
gh run view <id> --log-failed 2>&1 | grep -E "(\[error\]|FAIL|panic:|Error:|cannot |undefined:|syntax error)" | head -30
```

## Step 3: Get surrounding context if needed

If the filtered output isn't enough to diagnose, get more context around the error lines:
```bash
gh run view <id> --log-failed 2>&1 | grep -B5 -A5 "\[error\]" | head -60
```

Only dump the full `--log-failed` output as a last resort.

## Step 4: Analyze and suggest fixes

- Build/compilation errors → identify the file/line, run `/flash:test` locally to reproduce
- Test failures → run `/flash:test <package>` to reproduce the specific package
- Docker build failures → run `/flash:build` locally
- Deployment failures → check `/flash:prod-status` and `/flash:gcp-status`
- Multiple `main` declarations → add `//go:build ignore` to standalone scripts

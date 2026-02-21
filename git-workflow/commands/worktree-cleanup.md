---
description: "Remove stale/merged/gone worktrees — fetches remote, finds gone branches + merged branches + stale (>14 days), checks for open PRs before deleting, interactive selection"
argument-hint: "[all|stale|gone]"
allowed-tools: Bash(git *), Bash(gh *), Read, Glob, Grep, AskUserQuestion
---

## Context

- Git root: !`git rev-parse --show-toplevel`
- Remote default branch: !`git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}' || echo "master"`
- All worktrees: !`git worktree list`
- Branches with tracking status: !`git branch -vv`

## Your Task

Clean up stale, merged, and gone worktrees and their branches.

Arguments: $ARGUMENTS

Valid arguments:
- `all` — Remove all worktrees that are gone, merged, or stale (>14 days)
- `stale` — Only remove worktrees with no commits in >14 days
- `gone` — Only remove worktrees whose remote branch is gone

If no argument is provided, detect and present all candidates and let the user choose.

### 1. Fetch and prune remote

```bash
git fetch origin --prune
```

### 2. Identify cleanup candidates

Scan all worktrees (skip the main working tree) and categorize each:

**Gone branches** — remote tracking branch has been deleted:
```bash
git branch -vv | grep '\[.*: gone\]'
```

**Merged branches** — already merged into the base branch:
```bash
git branch --merged origin/<base-branch> | grep -v '^\*' | grep -v '<base-branch>'
```

**Stale branches** — last commit is older than 14 days:
```bash
# For each worktree branch
git log -1 --format="%ct" <branch-name>
# Compare with: date +%s
# If difference > 1209600 (14 days in seconds), it's stale
```

A worktree can appear in multiple categories (e.g., merged AND stale).

### 3. Check for open PRs

Before marking anything for deletion, check for open PRs:
```bash
gh pr list --head <branch-name> --json number,title,state --limit 1 2>/dev/null
```

If a branch has an open PR, flag it with a warning. Do NOT auto-delete branches with open PRs unless the user explicitly confirms.

### 4. Present candidates

If no argument was given, present the findings:

```
Worktree Cleanup Candidates
============================

GONE (remote branch deleted):
  [ ] .worktrees/feature-old — branch: feature/old (last commit: 3 weeks ago)

MERGED (already in <base-branch>):
  [ ] .worktrees/feature-done — branch: feature/done (last commit: 5 days ago)

STALE (no commits in >14 days):
  [ ] .worktrees/feature-abandoned — branch: feature/abandoned (last commit: 30 days ago)
      ⚠ Has open PR: #42 "Add feature" — will NOT auto-delete

No candidates: <category>

Which would you like to clean up?
```

Use AskUserQuestion to let the user select: all candidates, a specific category, or specific worktrees.

### 5. Execute cleanup

For each confirmed worktree to remove:

```bash
# Remove the worktree
git worktree remove <worktree-path>

# If that fails (dirty worktree), ask user whether to force
git worktree remove --force <worktree-path>

# Delete the local branch
git branch -D <branch-name>
```

If a worktree has uncommitted changes:
- Report the uncommitted changes
- Ask the user whether to force-remove or skip

### 6. Prune and report

```bash
git worktree prune
```

Report:
```
Cleanup complete:
  Removed: <N> worktrees
  Deleted: <N> branches
  Skipped: <N> (open PRs / uncommitted changes)

Remaining worktrees:
  <list remaining, if any>
```

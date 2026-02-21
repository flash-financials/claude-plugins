---
description: "Dashboard of all worktrees — branch, uncommitted changes, ahead/behind base, last commit, remote push status, open PR status, staleness warnings"
allowed-tools: Bash(git *), Bash(gh *), Read, Glob, Grep
---

## Context

- Git root: !`git rev-parse --show-toplevel`
- Current branch: !`git branch --show-current`
- Remote default branch: !`git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}' || echo "master"`
- All worktrees: !`git worktree list`

## Your Task

Display a comprehensive dashboard of all git worktrees in this repository.

### 1. Fetch latest remote state

```bash
git fetch origin --prune
```

### 2. Gather data for each worktree

For each worktree listed above (skip the main working tree — it's the bare repo root), collect:

**Branch info:**
```bash
# Branch name (from worktree list output)
# Last commit
git -C <worktree-path> log -1 --format="%h %s (%cr)"
```

**Uncommitted changes:**
```bash
git -C <worktree-path> status --short | wc -l
```

**Ahead/behind base branch:**
```bash
git rev-list --left-right --count origin/<base-branch>...<branch-name> 2>/dev/null
```
This outputs `<behind>\t<ahead>`. If the branch doesn't exist on remote yet, note "not pushed".

**Remote push status:**
```bash
git branch -r --list "origin/<branch-name>" 2>/dev/null
```
If output is empty, the branch hasn't been pushed.

**Open PR status:**
```bash
gh pr list --head <branch-name> --json number,title,state,url --limit 1 2>/dev/null
```

**Staleness** (days since last commit):
```bash
git -C <worktree-path> log -1 --format="%ct"
```
Compare with current timestamp. Warn if >14 days old.

### 3. Display the dashboard

Format the output as a clear table/report:

```
Git Worktree Dashboard
======================

Base branch: <base-branch>

<worktree-path>
  Branch:     <branch-name>
  Last commit: <hash> <message> (<time-ago>)
  Changes:    <N> uncommitted files
  vs base:    <N> ahead, <N> behind
  Remote:     pushed / not pushed
  PR:         #<number> <title> (<state>) — <url>
              or: no PR
  Staleness:  <N> days — WARNING: stale (>14 days)

<worktree-path-2>
  ...

Summary: <N> worktrees, <N> with uncommitted changes, <N> stale, <N> with open PRs
```

If there are no worktrees (besides the main working tree), report: "No worktrees found. Use `/git-workflow:worktree-start <description>` to create one."

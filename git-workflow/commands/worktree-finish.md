---
description: "Finish worktree work — commit remaining changes, run tests, then push+PR (default), merge locally, keep, or discard"
argument-hint: "[pr|merge|keep|discard]"
allowed-tools: Bash(git *), Bash(gh *), Bash(make *), Bash(go *), Bash(npm *), Bash(cargo *), Bash(pytest *), Read, Glob, Grep, AskUserQuestion
---

## Context

- Current branch: !`git branch --show-current`
- Git root: !`git rev-parse --show-toplevel`
- Working directory: !`pwd`
- In worktree: !`git rev-parse --git-common-dir 2>/dev/null`
- Worktree list: !`git worktree list 2>/dev/null`
- Git status: !`git status --short`
- Recent commits on this branch: !`git log --oneline -10`
- Remote default branch: !`git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}' || echo "master"`

## Your Task

Finish work on the current worktree branch.

Arguments: $ARGUMENTS

Valid arguments: `pr` (default), `merge`, `keep`, `discard`. If no argument is provided, default to `pr`.

### 1. Verify you're on a feature branch

Check that the current branch is NOT the base branch (master/main). If it is, tell the user they need to be on a feature branch and stop.

### 2. Commit any remaining changes

Check git status. If there are uncommitted changes:

- Analyze all changed files and group them into logical commits.
- Use conventional commit prefixes: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`.
- Write clear commit messages that explain the "why".
- Do NOT commit files that shouldn't be committed (`.env`, credentials, build artifacts, IDE files, OS files).
- If unsure about a file, ask the user.

If no uncommitted changes, skip this step.

### 3. Run tests

Auto-detect and run the project's test suite:

| File present | Command |
|---|---|
| `go.mod` | `go test ./...` |
| `package.json` | `npm test` |
| `Cargo.toml` | `cargo test` |
| `Makefile` with `test` target | `make test` |

If tests fail, report the failures and stop. Do NOT proceed with any finish action on broken code. Tell the user to fix tests first.

### 4. Execute the chosen action

#### `pr` (default) — Push and create a Pull Request

```bash
git push -u origin <branch-name>
```

Then create a PR with `gh pr create`:
- Target the base branch.
- Title: short, descriptive (under 70 chars), derived from branch name or commit messages.
- Body format:
  ```
  ## Summary
  <bullet points summarizing the changes>

  ## Test plan
  <how to verify the changes>
  ```

Report the PR URL to the user.

**Do NOT remove the worktree** — the user may need it for PR review feedback.

#### `merge` — Merge locally

```bash
# Get the worktree path before switching
WORKTREE_PATH=$(pwd)

# Switch to the main repo (not the worktree)
cd <git-root>  # The main repo root, not the worktree

# Update base branch
git checkout <base-branch>
git pull origin <base-branch>

# Merge the feature branch
git merge <branch-name>
```

Run tests again on the merged result. If tests fail, abort the merge and report.

If tests pass:
```bash
# Delete the feature branch
git branch -d <branch-name>

# Remove the worktree
git worktree remove <worktree-path>
```

Report: "Merged `<branch>` into `<base>`. Worktree cleaned up."

#### `keep` — Keep as-is

Report:
```
Keeping branch <name> and worktree at <path>.
Nothing was pushed or merged. You can resume work anytime.
```

Do nothing else.

#### `discard` — Discard all work

**ALWAYS ask for confirmation before discarding:**

```
This will permanently delete:
  - Branch: <branch-name>
  - Commits: <list recent commits>
  - Worktree: <path>

Are you sure you want to discard this work?
```

Use AskUserQuestion and only proceed if the user confirms.

If confirmed:
```bash
WORKTREE_PATH=$(pwd)
cd <git-root>
git worktree remove --force <worktree-path>
git branch -D <branch-name>
```

Report: "Discarded branch `<branch>` and removed worktree."

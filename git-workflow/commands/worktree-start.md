---
description: Create an isolated worktree for new work — derives branch name with smart prefix (feature/, fix/, docs/), ensures .worktrees/ is gitignored, creates worktree from latest base, installs deps, runs baseline tests
argument-hint: "<work-description>"
allowed-tools: Bash(git *), Bash(gh *), Bash(make *), Bash(go *), Bash(npm *), Bash(cargo *), Bash(pip *), Bash(poetry *), Bash(ls *), Bash(mkdir *), Bash(echo *), Read, Glob, Grep, AskUserQuestion
---

## Context

- Current branch: !`git branch --show-current`
- Git root: !`git rev-parse --show-toplevel`
- Remote default branch: !`git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}' || echo "master"`
- Existing worktrees: !`git worktree list 2>/dev/null`
- Git status: !`git status --short`

## Your Task

Create an isolated git worktree for feature development.

Arguments: $ARGUMENTS

The argument should be a short description of the feature (used for branch name). If not provided, ask the user what the feature is about.

### 1. Derive the branch name

Convert the argument to a kebab-case branch name with a smart prefix:

1. **Explicit prefix** — if the description already starts with `feature/`, `fix/`, or `docs/`, use it as-is.
2. **Auto-detect from keywords** — otherwise, check the first word(s):
   - `fix`, `bug`, `patch`, `hotfix` → `fix/`
   - `docs`, `document`, `readme`, `typo` → `docs/`
   - Everything else → `feature/`

Examples:
- `"fix login timeout"` → `fix/login-timeout`
- `"document API endpoints"` → `docs/api-endpoints`
- `"add user auth"` → `feature/add-user-auth`
- `"fix/cors-headers"` → `fix/cors-headers` (explicit, kept as-is)

### 2. Determine the base branch

Use the remote default branch detected above (typically `master` or `main`).

### 3. Update the base branch

```bash
git fetch origin <base-branch>
```

### 4. Ensure `.worktrees/` directory is gitignored

Check if `.worktrees/` is ignored:
```bash
git check-ignore -q .worktrees 2>/dev/null
echo $?
```

If NOT ignored (exit code non-zero):
1. Check if `.gitignore` exists at the repo root. If not, create it.
2. Append `.worktrees/` to `.gitignore`.
3. Commit the change:
   ```bash
   git add .gitignore
   git commit -m "chore: add .worktrees/ to gitignore"
   ```

### 5. Create the worktree

```bash
git worktree add .worktrees/<branch-name> -b <branch-name> origin/<base-branch>
```

This creates a new worktree at `.worktrees/<branch-name>` with a new branch tracking the latest base.

### 6. Install dependencies

`cd` into the worktree directory and auto-detect the project type:

| File present | Command |
|---|---|
| `go.mod` | `go mod download` |
| `package.json` | `npm install` |
| `Cargo.toml` | `cargo build` |
| `requirements.txt` | `pip install -r requirements.txt` |
| `pyproject.toml` | `poetry install` |

If none of these files exist, skip this step.

### 7. Run baseline tests

Run the project's test suite from the worktree directory to establish a clean baseline:

| File present | Command |
|---|---|
| `go.mod` | `go test ./...` |
| `package.json` | `npm test` |
| `Cargo.toml` | `cargo test` |
| `requirements.txt` or `pyproject.toml` | `pytest` |
| `Makefile` with `test` target | `make test` |

If tests fail, report the failures and ask the user whether to proceed or investigate. Do NOT silently continue.

If tests pass, continue.

### 8. Report

Report to the user:

```
Worktree created:
  Path:   <full-path-to-worktree>
  Branch: <branch-name>
  Base:   <base-branch> (up to date)
  Tests:  passing (<N> tests)

To work in this worktree, I'll operate from <full-path>.
```

Stay in the worktree directory for subsequent work. The user can now ask you to implement features in this isolated workspace.

---
description: Create a feature branch with well-structured commits, push, and open a PR. Use --worktree/-w to create in an isolated worktree instead.
argument-hint: "<feature-description> [--worktree|-w]"
allowed-tools: Bash(git *), Bash(gh *), Bash(make *), Bash(go *), Bash(npm *), Bash(cargo *), Bash(pip *), Bash(poetry *), Bash(ls *), Bash(mkdir *), Bash(echo *), Read, Glob, Grep, AskUserQuestion
---

## Context

- Current branch: !`git branch --show-current`
- Git status: !`git status --short`
- Remote default branch: !`git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}' || echo "master"`

## Your Task

Create a feature branch from a clean, up-to-date base branch with well-structured commits for all current changes. Push and open a PR.

Arguments: $ARGUMENTS

The argument should be a short description of the feature (used for branch name and PR title). If not provided, ask the user what the feature is about.

### Check for --worktree flag

If `$ARGUMENTS` contains `--worktree` or `-w`, use the **Worktree Flow** below instead of the default stash-based flow. Strip the flag from the arguments before deriving the branch name.

### 1. Determine the base branch

Use the remote default branch detected above (typically `master` or `main`).

### 2. Ensure working tree has changes

Check the git status from context above. If there are no changes at all, tell the user there's nothing to commit and stop.

### 3. Stash current work

Stash everything including untracked files:
```
git stash push --include-untracked -m "temp: staging for feature branch"
```

### 4. Update base branch

```
git checkout <base-branch> && git pull origin <base-branch>
```

### 5. Create the feature branch

Derive a branch name from the argument with a smart prefix:

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

```
git checkout -b <prefix>/<name>
```

### 6. Restore changes

Pop the stash:
```
git stash pop
```

Run `git status` to see all changes.

### 7. Create meaningful commits

DO NOT just `git add -A && git commit`. Instead:
- Analyze all the changed files and group them into logical commits.
- Each commit should represent a coherent unit of work.
- Use conventional commit prefixes: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`.
- Write clear commit messages that explain the "why", not just the "what".
- Be careful NOT to commit files that look like they shouldn't be committed:
  - `.env` files, credentials, secrets
  - Build artifacts (`node_modules/`, `bin/`, `dist/` unless intentional)
  - IDE files (`.idea/`, `.vscode/`)
  - OS files (`.DS_Store`, `Thumbs.db`)
  - Large binary files that look like working files (e.g. random .xlsx in root)
  - `tsconfig.tsbuildinfo`
- If unsure about a file, ask the user.

For example, if there are backend API changes and frontend UI changes, those could be separate commits:
```
feat: add integration selection to report service API
feat: add integration picker UI to create run page
```

### 8. Run tests

Run `make test && make lint` to verify nothing is broken.

If tests fail, report the failures and ask the user how to proceed. Do NOT push broken code.

### 9. Push the branch

```
git push -u origin feature/<name>
```

### 10. Create a Pull Request

Use `gh pr create` targeting the base branch.
- Title: short, descriptive (under 70 chars).
- Body format:

```
## Summary
<bullet points summarizing the changes>

## Test plan
<how to verify the changes>
```

Report the PR URL to the user.

---

## Worktree Flow (when `--worktree` or `-w` is used)

Instead of the stash-based flow above, create an isolated worktree. This keeps the current working tree untouched.

### W1. Determine the base branch

Same as step 1 above.

### W2. Ensure working tree has changes

Same as step 2 above.

### W3. Update base branch

```bash
git fetch origin <base-branch>
```

### W4. Derive the branch name

Same as step 5 above — smart prefix detection (`feature/`, `fix/`, or `docs/`) + kebab-case description.

### W5. Ensure `.worktrees/` is gitignored

```bash
git check-ignore -q .worktrees 2>/dev/null
```

If NOT ignored, add `.worktrees/` to `.gitignore` and commit.

### W6. Create the worktree

```bash
git worktree add .worktrees/<branch-name> -b <branch-name> origin/<base-branch>
```

### W7. Move changes to the worktree

Stash the current changes, then pop them in the worktree:
```bash
git stash push --include-untracked -m "temp: moving to worktree"
cd .worktrees/<branch-name>
git stash pop
```

### W8. Create meaningful commits

Same as step 7 above — analyze, group, and commit logically.

### W9. Run tests

Same as step 8 above.

### W10. Push and create PR

Same as steps 9 and 10 above.

Report the PR URL and worktree location to the user.

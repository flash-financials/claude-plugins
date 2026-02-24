## Git Workflow

Always use worktrees when starting new work. This keeps the main working tree clean and allows safe multi-tasking.

- **Start:** Use the `superpowers:using-git-worktrees` skill to create an isolated worktree workspace
- **Finish:** Use the `superpowers:finishing-a-development-branch` skill to merge, PR, keep, or discard the branch

Worktree directory preference: `.worktrees/` (project-local, hidden)

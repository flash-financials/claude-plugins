## Git Workflow

**Never push directly to main.** Always use worktrees and feature branches.

- **Start:** `git worktree add .worktrees/<name> -b feature/<name>` or use `superpowers:using-git-worktrees`
- **Finish:** Push branch, open PR via `gh pr create`, merge to main. Or use `superpowers:finishing-a-development-branch`
- **Cleanup:** `git worktree remove .worktrees/<name>` after merge

Worktree directory: `.worktrees/` (gitignored)

## Plugin Cache Sync

After editing source files, copy them to the local cache for immediate effect:

```bash
cp -r flash/agents/*.md ~/.claude/plugins/cache/local/flash/1.0.0/agents/
cp -r flash/skills/*.md ~/.claude/plugins/cache/local/flash/1.0.0/skills/
```

Cache changes are local-only. Merge PR to main to persist across machines.

---
name: ci-status
description: Check GitHub Actions CI/CD workflow status
allowed-tools: Bash(gh *)
---

Check recent GitHub Actions workflow runs for this repository.

```bash
gh run list --limit 10
```

Show the results in a readable table. Highlight any failures or in-progress runs.

If there are failures, suggest running `/flash:ci-logs` to investigate the failed run, or `/flash:test` to reproduce locally.

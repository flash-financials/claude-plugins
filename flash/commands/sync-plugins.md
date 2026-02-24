---
description: Sync local flash plugin to the claude-plugins GitHub repo
disable-model-invocation: true
allowed-tools: Bash(git *), Bash(gh *), Bash(rm *), Bash(cp *), Bash(mkdir *), Bash(ls *), Bash(diff *), Bash(cat *), Read, AskUserQuestion
---

Sync the local flash plugin to the `flash-financials/claude-plugins` GitHub repo.

**Local plugin path:** `~/.claude/plugins/cache/local/flash/1.0.0/`
**Local marketplace:** `~/.claude/plugins/marketplaces/local/.claude-plugin/marketplace.json`
**Remote repo:** `flash-financials/claude-plugins` (branch: `main`)

## Steps

1. **Clone the repo to a temp directory:**
   ```bash
   rm -rf /tmp/claude-plugins-sync
   git clone https://github.com/flash-financials/claude-plugins.git /tmp/claude-plugins-sync
   ```

2. **Replace flash/ with local version:**
   ```bash
   rm -rf /tmp/claude-plugins-sync/flash/
   cp -R ~/.claude/plugins/cache/local/flash/1.0.0/ /tmp/claude-plugins-sync/flash/
   ```

3. **Update root marketplace.json from local:**
   ```bash
   cp ~/.claude/plugins/marketplaces/local/.claude-plugin/marketplace.json /tmp/claude-plugins-sync/marketplace.json
   ```

4. **Remove any leftover old plugin directories** (these were consolidated into flash):
   ```bash
   rm -rf /tmp/claude-plugins-sync/ssh-tunnel/ /tmp/claude-plugins-sync/skill-authoring/ /tmp/claude-plugins-sync/git-workflow/ /tmp/claude-plugins-sync/terraform/
   ```

5. **Show the diff and ask user to confirm:**
   ```bash
   cd /tmp/claude-plugins-sync && git status --short && echo "---" && git diff --stat
   ```
   If there are no changes, report "Already in sync — nothing to push." and skip to step 8.

   Otherwise, show the summary and ask the user: "Push these changes to flash-financials/claude-plugins?"

6. **Commit with a descriptive message:**
   ```bash
   cd /tmp/claude-plugins-sync && git add -A && git commit -m "Sync flash plugin from local"
   ```

7. **Push to remote:**
   ```bash
   cd /tmp/claude-plugins-sync && git push origin main
   ```

8. **Clean up temp directory:**
   ```bash
   rm -rf /tmp/claude-plugins-sync
   ```

Report the result: either "Synced successfully" with the commit hash, or "Already in sync."

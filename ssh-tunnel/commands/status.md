---
description: Check if an SSH tunnel is active on a given port
argument-hint: "[port]"
allowed-tools: Bash(lsof *), Bash(cat *), Read
---

## Context

- Existing tunnel config: !`cat .tunnel-config.json 2>/dev/null || echo "No .tunnel-config.json found"`

## Your Task

Check whether an SSH tunnel is currently active on the specified port.

### Step 1: Determine Port

If a port argument was provided, use it. Otherwise, read the `localPort` from `.tunnel-config.json`. If neither is available, ask the user which port to check.

### Step 2: Check Status

```bash
lsof -i :<PORT> -sTCP:LISTEN
```

### Step 3: Report

- If a process is found: report the tunnel is **active**, show the PID and process details. Remind the user they can close it with `/ssh-tunnel:close <PORT>`.
- If no process is found: report **no active tunnel** on that port. Suggest `/ssh-tunnel:open` to create one.

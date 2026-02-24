---
name: tunnel-close
description: Close an active SSH tunnel on a given port
argument-hint: "[port]"
allowed-tools: Bash(lsof *), Bash(kill *), Bash(cat *), Read
---

## Context

- Existing tunnel config: !`cat .tunnel-config.json 2>/dev/null || echo "No .tunnel-config.json found"`

## Your Task

Close an active SSH tunnel by killing the process listening on the specified port.

### Step 1: Determine Port

If a port argument was provided, use it. Otherwise, read the `localPort` from `.tunnel-config.json`. If neither is available, ask the user which port to close.

### Step 2: Find the Process

```bash
lsof -i :<PORT> -sTCP:LISTEN
```

If no process is found, inform the user that no tunnel is active on that port.

### Step 3: Kill the Process

```bash
kill $(lsof -t -i :<PORT> -sTCP:LISTEN)
```

### Step 4: Verify

Run `lsof -i :<PORT> -sTCP:LISTEN` again to confirm the process is gone. Report success or failure.

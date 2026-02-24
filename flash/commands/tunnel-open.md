---
description: Open an SSH tunnel through a GCP VPS to a remote endpoint
argument-hint: "[project vps zone local_port remote_host remote_port]"
allowed-tools: Bash(gcloud *), Bash(lsof *), Bash(kill *), Bash(brew *), Bash(which *), Bash(pwd), Bash(cat *), Read, Write, AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Existing tunnel config: !`cat .tunnel-config.json 2>/dev/null || echo "No .tunnel-config.json found"`
- gcloud installed: !`which gcloud 2>/dev/null || echo "NOT INSTALLED"`
- gcloud active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`

## Your Task

Open an SSH tunnel through a GCP Compute Engine VPS to a remote endpoint.

### Step 1: Check Prerequisites

- If gcloud is NOT INSTALLED, tell the user to run `brew install google-cloud-sdk` and restart their shell. Stop here.
- If gcloud is NOT AUTHENTICATED, tell the user to run `gcloud init && gcloud auth login`. Stop here.

### Step 2: Determine Parameters

If arguments were provided to this command, parse them as positional: `project vps zone local_port remote_host remote_port`.

Otherwise, check the tunnel config from context above. If no config exists either, ask the user for all 6 parameters:
- **project**: GCP project ID
- **vps**: Compute Engine instance name
- **zone**: GCP zone
- **localPort**: Local port to listen on
- **remoteHost**: Remote host accessible from the VPS
- **remotePort**: Remote port to forward to

### Step 3: Check for Existing Tunnel

Run `lsof -i :<LOCAL_PORT> -sTCP:LISTEN` to check if the port is already in use. If so, inform the user and ask whether to kill the existing process or use a different port.

### Step 4: Open the Tunnel

```bash
gcloud config set project <PROJECT>
gcloud compute ssh <VPS> --zone=<ZONE> -- -L <LOCAL_PORT>:<REMOTE_HOST>:<REMOTE_PORT> -N -f
```

### Step 5: Verify

Wait 2 seconds, then run:
```bash
lsof -i :<LOCAL_PORT> -sTCP:LISTEN
```

Report success or failure. On success, show:
- Local endpoint: `localhost:<LOCAL_PORT>`
- To close: run `/flash:tunnel-close <LOCAL_PORT>` or `kill $(lsof -t -i :<LOCAL_PORT> -sTCP:LISTEN)`

### Step 6: Save Config

If no `.tunnel-config.json` existed, offer to save the parameters for future use.

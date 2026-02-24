---
description: View or edit the SSH tunnel configuration for this project
argument-hint: "[key value]"
allowed-tools: Bash(pwd), Bash(cat *), Read, Write, AskUserQuestion
---

## Context

- Current directory: !`pwd`
- Existing tunnel config: !`cat .tunnel-config.json 2>/dev/null || echo "No .tunnel-config.json found"`

## Your Task

View or edit the `.tunnel-config.json` file in the current project root.

### If no arguments provided

If a config file exists, display the current configuration in a readable format. If no config exists, ask the user if they want to create one.

### If arguments provided

Parse arguments as `key value` pairs to update specific fields. Valid keys:
- **project**: GCP project ID
- **vps**: Compute Engine instance name
- **zone**: GCP zone (e.g., us-central1-a)
- **localPort**: Local port to listen on (e.g., 5433)
- **remoteHost**: Remote host IP/hostname accessible from the VPS
- **remotePort**: Remote port to connect to (e.g., 5432)

### Creating a new config

If no `.tunnel-config.json` exists and the user wants to create one, ask for all 6 parameters and write the file:

```json
{
  "project": "<PROJECT>",
  "vps": "<VPS>",
  "zone": "<ZONE>",
  "localPort": <LOCAL_PORT>,
  "remoteHost": "<REMOTE_HOST>",
  "remotePort": <REMOTE_PORT>
}
```

### Editing existing config

Read the current config, update the specified key(s), and write it back. Show the user what changed.

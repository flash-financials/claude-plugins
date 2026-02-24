---
name: ssh-tunnel-skill
description: Open SSH tunnels using a GCP VM as a VPS jump host to reach remote endpoints. Use when the user mentions SSH tunneling, connecting through a VPS, port forwarding, Cloud SQL access, or creating a tunnel to a remote host.
---

This skill helps users create and manage SSH tunnels using a GCP Compute Engine VM as a VPS jump host to access remote endpoints (databases, APIs, internal services).

## Prerequisites Check

Before opening a tunnel, verify these in order:

1. **gcloud CLI installed**: Run `which gcloud`. If not found, instruct user to install with `brew install google-cloud-sdk` and restart their shell.
2. **gcloud authenticated**: Run `gcloud auth list`. If no active account, instruct user to run `gcloud init && gcloud auth login`.

## Tunnel Configuration

Check for a `.tunnel-config.json` file in the current project root. If it exists, read it and use those values as defaults.

The config format:
```json
{
  "project": "my-gcp-project-id",
  "vps": "my-vps-instance-name",
  "zone": "us-central1-a",
  "localPort": 5433,
  "remoteHost": "10.0.0.1",
  "remotePort": 5432
}
```

If no config file exists, ask the user for all 6 parameters at once using AskUserQuestion or by requesting them in a single prompt:
- **project**: GCP project ID
- **vps**: Compute Engine instance name
- **zone**: GCP zone (e.g., us-central1-a)
- **localPort**: Local port to listen on (e.g., 5433)
- **remoteHost**: Remote host IP/hostname accessible from the VPS
- **remotePort**: Remote port to connect to (e.g., 5432)

## Opening the Tunnel

1. Set the GCP project:
   ```
   gcloud config set project <PROJECT>
   ```

2. Open the SSH tunnel in the background:
   ```
   gcloud compute ssh <VPS> --zone=<ZONE> -- -L <LOCAL_PORT>:<REMOTE_HOST>:<REMOTE_PORT> -N -f
   ```

3. Wait a few seconds, then verify the tunnel is active:
   ```
   lsof -i :<LOCAL_PORT> -sTCP:LISTEN
   ```

4. If verification succeeds, report success and show the user:
   - The local endpoint: `localhost:<LOCAL_PORT>`
   - How to close the tunnel: `kill $(lsof -t -i :<LOCAL_PORT> -sTCP:LISTEN)` or `/flash:tunnel-close`

5. If no config file existed, offer to save the configuration to `.tunnel-config.json` in the project root for future use.

## Important Notes

- The `-N` flag means no remote command is executed (tunnel only).
- The `-f` flag backgrounds the SSH process.
- If the port is already in use, check if an existing tunnel is running with `lsof -i :<PORT> -sTCP:LISTEN` and inform the user.
- If the SSH connection fails, suggest the user check:
  - VPS instance is running: `gcloud compute instances describe <VPS> --zone=<ZONE> --format='value(status)'`
  - Firewall rules allow SSH: `gcloud compute firewall-rules list --filter="allowed[].ports:22"`
  - OS Login or SSH key issues: `gcloud compute ssh <VPS> --zone=<ZONE>` (without tunnel flags, to test basic connectivity)

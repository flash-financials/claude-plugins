# GCP SSH Tunnel Plugin

A Claude Code plugin for managing SSH tunnels through GCP Compute Engine VPS instances to access remote endpoints (databases, APIs, internal services).

## Commands

| Command | Description |
|---------|-------------|
| `/tunnel-open [project vps zone local_port remote_host remote_port]` | Open an SSH tunnel through a GCP VPS |
| `/tunnel-close [port]` | Close an active SSH tunnel |
| `/tunnel-status [port]` | Check if a tunnel is active |

## Auto-Trigger

The SSH tunnel skill also activates automatically when you mention SSH tunneling, connecting through a VPS, port forwarding, or Cloud SQL access.

## Configuration

Create a `.tunnel-config.json` in your project root to save defaults:

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

## Prerequisites

- **macOS** with Homebrew
- **Google Cloud SDK**: `brew install google-cloud-sdk`
- **Authentication**: `gcloud init && gcloud auth login`

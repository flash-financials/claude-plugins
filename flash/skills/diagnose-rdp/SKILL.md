---
name: diagnose-rdp
description: Use when RDP connection to the GCP workstation fails, is refused, hangs,
  or drops. Also use when xrdp errors appear in logs, the desktop is unreachable after
  VM start, or the user reports "can't connect" to the remote desktop.
---

# Diagnose RDP Connection Issues

## Environment Constants

- **Project:** `infra-admin-355646`
- **Zone:** `us-west1-a`
- **VM name:** `workstation`
- **Static IP:** `34.127.104.53`
- **RDP port:** `3389`
- **Desktop user:** `admin`
- **Config:** `/etc/xrdp/xrdp.ini`

## Diagnostic Checklist

Run these in order. SSH in via IAP (works regardless of firewall state):

```bash
SSH="gcloud compute ssh workstation --zone=us-west1-a --project=infra-admin-355646 --tunnel-through-iap --command"
```

### 1. VM Running?

```bash
gcloud compute instances describe workstation --zone=us-west1-a --project=infra-admin-355646 --format="value(status)"
```

Expected: `RUNNING`. If not, start it and wait 30s for xrdp.

### 2. xrdp Service Active?

```bash
$SSH "systemctl is-active xrdp; systemctl is-active xrdp-sesman"
```

Expected: both `active`. If not: `$SSH "sudo systemctl restart xrdp"`

### 3. Listening on Port 3389?

```bash
$SSH "sudo ss -tlnp | grep 3389"
```

Expected: `LISTEN *:3389` by xrdp process. If missing, xrdp failed to bind — check logs.

### 4. Backend Session Ports (CRITICAL)

```bash
$SSH "grep -n 'port=' /etc/xrdp/xrdp.ini | grep -v '^[0-9]*:;'"
```

**Expected:**
- `[Globals]` section: `port=3389` (listening port)
- `[Xorg]` section: `port=-1` (auto-assign)
- `[Xvnc]` section: `port=-1` (auto-assign)

**Known bug:** A bad `sed` in the startup script can overwrite ALL `port=` lines to `3389`, causing xrdp to connect back to itself instead of the display server. Symptoms in logs: `libxrdp_force_read: bad header length 0` and `Connection Sequence: CR-TPDU failed`.

**Fix:**

```bash
$SSH "sudo sed -i '/^\[Xorg\]/,/^\[/{s/^port=.*/port=-1/}' /etc/xrdp/xrdp.ini && sudo sed -i '/^\[Xvnc\]/,/^\[/{s/^port=.*/port=-1/}' /etc/xrdp/xrdp.ini && sudo systemctl restart xrdp"
```

### 5. TLS Key Permissions

```bash
$SSH "getent group ssl-cert"
```

Expected: `ssl-cert:x:...:xrdp`. If xrdp is not in the group:

```bash
$SSH "sudo adduser xrdp ssl-cert && sudo systemctl restart xrdp"
```

### 6. xrdp Logs

```bash
$SSH "sudo tail -40 /var/log/xrdp.log"
$SSH "sudo tail -20 /var/log/xrdp-sesman.log"
```

| Log Pattern | Meaning |
|-------------|---------|
| `bad header length 0` | Backend port misconfigured (see step 4) |
| `CR-TPDU failed` | xrdp connecting to itself (see step 4) |
| `cannot read private key` | TLS permission issue (see step 5) |
| `Login failed for user` | Wrong password — check `/home/admin/.rdp_password` |

### 7. Firewall (UFW)

```bash
$SSH "sudo ufw status numbered"
```

Verify port 3389 is allowed from the user's IP or CIDR. Add if missing:

```bash
$SSH "sudo ufw allow from <CIDR> to any port 3389 proto tcp"
```

### 8. GCP Firewall

```bash
gcloud compute firewall-rules list --project=infra-admin-355646 --filter="name:rdp OR name:workstation" --format="table(name,direction,sourceRanges,allowed)"
```

Verify the user's IP is in the source ranges for the RDP allow rule.

### 9. IAP Tunnel Fallback

If firewall issues can't be resolved quickly, tunnel RDP through IAP:

```bash
gcloud compute start-iap-tunnel workstation 3389 --local-host-port=localhost:3389 --zone=us-west1-a --project=infra-admin-355646
```

Then connect Microsoft Remote Desktop to `localhost:3389`.

## Startup Script Fix

If the backend port bug recurs after VM restarts, the startup script (`startup.sh` in `flash-financials/gcloud-workstation`) needs these lines after the listening port sed:

```bash
sed -i '/^\[Xorg\]/,/^\[/{s/^port=.*/port=-1/}' /etc/xrdp/xrdp.ini
sed -i '/^\[Xvnc\]/,/^\[/{s/^port=.*/port=-1/}' /etc/xrdp/xrdp.ini
```

---
name: gcloud-workstation
description: Start, stop, and check status of the remote GCP Linux desktop workstation.
  Use when the user mentions "workstation", "remote desktop", "start the VM", "stop
  the VM", or wants to connect via RDP.
---

# GCP Linux Desktop Workstation

## Context

- VM status: !`gcloud compute instances describe workstation --zone=us-west1-a --project=infra-admin-355646 --format="value(status)" 2>/dev/null || echo "UNKNOWN"`
- Static IP: `34.127.104.53`

## Environment Constants

- **Project:** `infra-admin-355646`
- **Zone:** `us-west1-a`
- **VM name:** `workstation`
- **Static IP:** `34.127.104.53`
- **RDP port:** `3389`
- **Desktop user:** `admin`
- **Repo:** `flash-financials/gcloud-workstation`

---

## Start

```bash
gcloud compute instances start workstation --zone=us-west1-a --project=infra-admin-355646
```

After starting, wait ~30 seconds for xrdp to come up, then verify:

```bash
gcloud compute ssh workstation --zone=us-west1-a --project=infra-admin-355646 --tunnel-through-iap --command="systemctl is-active xrdp && echo 'RDP ready'" 2>&1
```

Print RDP connection info after start:
- Open **Microsoft Remote Desktop** on Mac
- Connect to `34.127.104.53`
- User: `admin`
- Password: SSH in and run `sudo cat /home/admin/.rdp_password`

---

## Stop

```bash
gcloud compute instances stop workstation --zone=us-west1-a --project=infra-admin-355646
```

The static IP is preserved across stop/start cycles. Stopping saves ~$100/month compute costs.

---

## Status

```bash
gcloud compute instances describe workstation --zone=us-west1-a --project=infra-admin-355646 --format="table(name,status,networkInterfaces[0].accessConfigs[0].natIP,machineType.basename())"
```

---

## SSH

Direct (from whitelisted IP):
```bash
gcloud compute ssh workstation --zone=us-west1-a --project=infra-admin-355646
```

Via IAP tunnel (from any IP):
```bash
gcloud compute ssh workstation --zone=us-west1-a --project=infra-admin-355646 --tunnel-through-iap
```

---

## Whitelisted IPs

| Name | CIDR |
|------|------|
| lance | 104.53.182.67/32 |
| scott-vpn | 154.16.199.21/32 |
| flash-vps | 34.134.139.60/32 |

If user's IP isn't whitelisted, use IAP tunnel for SSH or tunnel RDP through IAP:

```bash
gcloud compute start-iap-tunnel workstation 3389 --local-host-port=localhost:3389 --zone=us-west1-a --project=infra-admin-355646
```

Then connect Microsoft Remote Desktop to `localhost:3389`.

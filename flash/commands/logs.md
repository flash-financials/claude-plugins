---
description: Show recent server logs
allowed-tools: Bash(tail *)
---

Show the accounting server logs.

Run `tail -100 /tmp/accounting-server.log` to show the last 100 lines.

If the user wants continuous/live logs, tell them to run `tail -f /tmp/accounting-server.log` in a separate terminal.

---
description: Stop the development server
allowed-tools: Bash(lsof *)
---

Stop the development server and verify it's down.

Steps:
1. Kill the server process:
   ```bash
   lsof -ti :8080 | xargs kill -9 2>/dev/null && echo "Server stopped" || echo "Server not running"
   ```
2. Verify: `lsof -ti :8080 >/dev/null 2>&1` — should report nothing.

Do NOT stop Docker/PostgreSQL unless the user explicitly asks.

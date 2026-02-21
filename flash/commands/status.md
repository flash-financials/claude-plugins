---
description: Check the health of all local services
allowed-tools: Bash(docker inspect *), Bash(lsof *), Bash(curl *)
---

Check the health of all local services. Run these three checks:

1. **PostgreSQL:** `docker inspect -f '{{.State.Status}}' accounting-db 2>/dev/null || echo "not running"`
2. **Server:** `lsof -ti :8080 >/dev/null 2>&1 && echo "running (pid $(lsof -ti :8080 | head -1))" || echo "not running"`
3. **Health:** `curl -sf http://localhost:8080/healthz >/dev/null 2>&1 && echo "healthy" || echo "unreachable"`

Report results clearly. If anything is down, suggest:
- PostgreSQL down → `/flash:dev`
- Server down → `/flash:dev`
- Health check failing → `/flash:logs` to check for errors

---
name: dev
description: Start the full local dev environment (Docker + server + health check)
allowed-tools: Bash(colima *), Bash(docker-compose *), Bash(docker inspect *), Bash(lsof *), Bash(DATABASE_URL=*), Bash(curl *), Bash(sleep *)
---

Start the full local development environment (PostgreSQL + server) and confirm everything is healthy.

Steps:

1. **Start Docker (colima + PostgreSQL)**
   ```bash
   colima status >/dev/null 2>&1 || colima start
   ```
   ```bash
   docker-compose up -d
   ```
   If colima fails to start, try `colima stop --force && colima start`.

2. **Kill any stale server and start fresh**
   ```bash
   lsof -ti :8080 | xargs kill -9 2>/dev/null || true
   ```
   ```bash
   DATABASE_URL="host=localhost port=6432 user=postgres password=123456 dbname=accounting sslmode=disable" \
   LISTEN_ADDR=":8080" \
   STATIC_DIR="frontend/dist" \
   GSHEET_CREDENTIALS="client_secret_122638021932-l9f9mrud9afam587citrdgrm4emd9l3s.apps.googleusercontent.com.json" \
   GSHEET_FOLDER="1DzVFWHuwz1oS4FyZ388XUzIBWbUHPZxk" \
   CHROME_PROFILE="Work" \
   SEED_ON_EMPTY=true \
   go run ./cmd/server > /tmp/accounting-server.log 2>&1 &
   ```

3. **Wait and verify**
   ```bash
   sleep 3
   ```
   Then check all three:
   - PostgreSQL: `docker inspect -f '{{.State.Status}}' accounting-db 2>/dev/null || echo "not running"`
   - Server: `lsof -ti :8080 >/dev/null 2>&1 && echo "running" || echo "not running"`
   - Health: `curl -sf http://localhost:8080/healthz >/dev/null 2>&1 && echo "healthy" || echo "unreachable"`

Report the final status. Common problems:
- Port 8080 in use: the kill step should handle it, but check if issues persist.
- Colima not starting: `colima stop --force && colima start`.
- PostgreSQL unhealthy: check `docker logs accounting-db`.

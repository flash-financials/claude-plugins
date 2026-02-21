---
description: Restart the full dev environment from scratch
allowed-tools: Bash(colima *), Bash(docker-compose *), Bash(docker inspect *), Bash(lsof *), Bash(DATABASE_URL=*), Bash(curl *), Bash(sleep *)
---

Restart the full development environment from scratch.

Steps:

1. **Stop existing server**
   ```bash
   lsof -ti :8080 | xargs kill -9 2>/dev/null || true
   ```

2. **Ensure Docker + PostgreSQL are running**
   ```bash
   colima status >/dev/null 2>&1 || colima start
   ```
   ```bash
   docker-compose up -d
   ```

3. **Start server fresh**
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

4. **Wait and verify**
   ```bash
   sleep 3
   ```
   Check: PostgreSQL (`docker inspect -f '{{.State.Status}}' accounting-db`), server (`lsof -ti :8080`), health (`curl -sf http://localhost:8080/healthz`).

Report the final status.

---
description: Check & fix gcloud authentication (login, ADC, Docker registry)
allowed-tools: Bash(gcloud *), Bash(cat *)
argument-hint: "[fix]"
---

Check gcloud authentication status and fix any issues.

## Context
- Active account: !`gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null || echo "NOT AUTHENTICATED"`
- Current project: !`gcloud config get-value project 2>/dev/null || echo "NOT SET"`
- ADC status: !`gcloud auth application-default print-access-token 2>&1 | head -1`
- Docker registry: !`cat ~/.docker/config.json 2>/dev/null | grep "docker.pkg.dev" || echo "NOT CONFIGURED"`

## Behavior

Arguments: $ARGUMENTS

**Step 1: Present an auth dashboard from the context above**

Evaluate each check and present a dashboard:

```
gcloud Auth Dashboard
======================
Account:         [OK]  user@example.com        | [FAIL] NOT AUTHENTICATED
Project:         [OK]  my-project-id           | [FAIL] NOT SET
ADC (app creds): [OK]  ya29.xxx...             | [FAIL] error/expired message
Docker Registry: [OK]  docker.pkg.dev helper   | [WARN] NOT CONFIGURED
```

Rules for status:
- **Account**: `[OK]` if a valid email is shown, `[FAIL]` if "NOT AUTHENTICATED"
- **Project**: `[OK]` if a project ID is shown, `[FAIL]` if "NOT SET"
- **ADC**: `[OK]` if the output starts with `ya29.`, `[FAIL]` otherwise (expired, error, etc.)
- **Docker Registry**: `[OK]` if `docker.pkg.dev` found in docker config, `[WARN]` if "NOT CONFIGURED"

**Step 2: Fix issues (if `$ARGUMENTS` is `fix` or any check failed)**

If arguments contain `fix`, or any check shows `[FAIL]` or `[WARN]`, run the appropriate fixes:

1. **Account FAIL** → run:
   ```bash
   gcloud auth login
   ```

2. **Project FAIL** → ask the user which project to set, then:
   ```bash
   gcloud config set project <PROJECT_ID>
   ```

3. **ADC FAIL** → run:
   ```bash
   gcloud auth application-default login
   ```

4. **Docker Registry WARN** → detect the region from the current project, then:
   ```bash
   gcloud auth configure-docker <REGION>-docker.pkg.dev
   ```
   To detect region:
   ```bash
   gcloud config get-value compute/region 2>/dev/null || echo "us-central1"
   ```

**Step 3: Re-check after fixes**

After running any fixes, re-run the checks to confirm everything is `[OK]`.

If everything is already `[OK]` and no `fix` argument was given, just report the healthy status.

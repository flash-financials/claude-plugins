---
description: Generate a dev JWT access token for API testing
allowed-tools: Bash(go run *)
---

Generate a development JWT access token for testing authenticated API endpoints.

Run `go run ./scripts/gen-token.go` which issues an admin-level JWT token using a hardcoded dev secret.

Print the token so the user can copy it for use in API requests (e.g., `Authorization: Bearer <token>`).

Note: This token uses a hardcoded dev secret and is only valid for local development. It will NOT work against production.

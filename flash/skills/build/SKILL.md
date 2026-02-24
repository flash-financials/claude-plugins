---
name: build
description: Build the Go CLI binary and React frontend
allowed-tools: Bash(cd frontend *), Bash(CGO_ENABLED=*)
---

Build the Go CLI binary and React frontend.

Steps:
1. **Frontend:** `cd frontend && npm ci && npm run build` (TypeScript check + Vite production build)
2. **Backend:** `CGO_ENABLED=0 go build -o bin/accounting ./cmd/reports`

Report success or failure. If the frontend build fails, check for TypeScript errors. If the Go build fails, check for compilation errors.

---
name: flyway
description: Flyway migration rules — naming, versioning, safety constraints
paths:
  - "**/db/migration/**"
  - "**/*Migration*"
  - "**/*migration*"
---

# Flyway Migration

**Directory:** `{{BACKEND_PATH}}/src/main/resources/db/migration/`

## Naming Format

```
V{major}.{minor}.{patch}__<snake_case_description>.sql
```

Check latest migration: `ls {{BACKEND_PATH}}/src/main/resources/db/migration/ | sort -V | tail -3`

## Rules

1. **Sequential and monotonically increasing.** Next number = last + 1.
2. **Never modify an existing migration** — checksum errors on deployed environments.
3. **Write idempotently:** `CREATE TABLE IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`.
4. **One logical change per migration.**
5. **Document rollback** (as header comment) if not obvious.

## Local Test

```bash
docker compose up -d postgres
{{BACKEND_BUILD_TOOL}} bootRun
# Logs: "Migrating schema to version..."
```

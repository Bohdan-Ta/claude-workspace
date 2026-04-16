---
name: workflow
description: Project-specific workflows — build commands, database migrations, API client regeneration, i18n process, PR review, Git operations.
---

# Workflow — Project Processes

Concrete step-by-step guides for recurring tasks.

For Claude behavior rules (Approval, KISS, Bug-First etc.) see `claude-behavior.md`.

---

## Quick Reference: `/review-pr`

```
/review-pr [PR#|URL]   # specific PR
/review-pr             # current branch vs. origin/{{DEFAULT_BRANCH}}
```

Starts relevant reviewer agents in parallel, aggregates findings by severity. Details: `.claude/skills/review-pr/SKILL.md`.

---

<!-- SPRING-BOOT -->
## Adding a New Module

Reference: `{{BACKEND_PATH}}/src/main/kotlin/{{PACKAGE_PATH}}/` as template.

### Module Structure

```
<module>/
├── <Module>Module.kt      — @ApplicationModule declaration
├── api/                   — Public Facade + DTOs (cross-module interface)
├── model/                 — JPA Entities + domain objects
├── mapper/                — DTO mappers
├── persistence/           — JPA Repositories
├── service/               — Business Logic
└── web/                   — REST Controllers
```

### Steps

1. **Create directory:** `{{BACKEND_PATH}}/src/main/kotlin/{{PACKAGE_PATH}}/<name>/` + subdirectories.

2. **Module declaration** — `<Name>Module.kt`:
   ```kotlin
   package {{PACKAGE_ROOT}}.<name>

   import org.springframework.modulith.ApplicationModule

   @ApplicationModule(
       allowedDependencies = [
           "shared :: service",
           "shared :: web",
           // add only what's actually needed
       ]
   )
   class <Name>Module
   ```

3. **Validation:**
   ```bash
   {{BACKEND_BUILD_TOOL}} validateModulePlacement
   ```

4. **Build + Tests:**
   ```bash
   {{BACKEND_BUILD_TOOL}} check
   ```

### Principles

- **Minimal dependencies** — declare each module dependency explicitly.
- **Facade for cross-module:** Module B accesses Module A only via `A :: api` (Facade Pattern).
- **Event-driven when possible** — for fire-and-forget (e.g., email triggers): publish Application Event instead of calling service.

---

## Database Migration (Flyway)

**Directory:** `{{BACKEND_PATH}}/src/main/resources/db/migration/`

**Check latest migration:**
```bash
ls {{BACKEND_PATH}}/src/main/resources/db/migration/ | sort -V | tail -3
```

### Naming Format

```
V{major}.{minor}.{patch}__<snake_case_description>.sql
```

### Rules

1. **Sequential and monotonically increasing.** Next number = last + 1.
2. **Never modify an existing migration** — checksum errors on deployed environments.
3. **Write idempotently where possible:** `CREATE TABLE IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`.
4. **One logical change per migration.**
5. **Document rollback** (as header comment) if not obvious.

### Local Test

```bash
docker compose up -d postgres
{{BACKEND_BUILD_TOOL}} bootRun
# Check logs: "Migrating schema to version..."
```
<!-- /SPRING-BOOT -->

<!-- NEXTJS -->
## Database Migration (Prisma)

**Schema location:** `prisma/schema.prisma`

### Workflow

1. **Edit schema** — add/modify models in `prisma/schema.prisma`
2. **Create migration:**
   ```bash
   npx prisma migrate dev --name <snake_case_description>
   ```
3. **Apply in production:**
   ```bash
   npx prisma migrate deploy
   ```

### Rules

1. **Never edit existing migration files** — they are immutable once applied.
2. **One logical change per migration.**
3. **Reset dev DB if needed:** `npx prisma migrate reset` (destroys all data).
4. **Generate client after schema changes:** `npx prisma generate`

---

## API Route Patterns

### File-Based API Routes

```
app/api/
├── events/
│   ├── route.ts              — GET (list), POST (create)
│   └── [id]/
│       ├── route.ts          — GET (detail), PUT (update), DELETE
│       └── register/
│           └── route.ts      — POST (action)
├── users/
│   └── route.ts
└── webhooks/
    └── stripe/
        └── route.ts
```

### Server Actions (alternative to API routes)

For form submissions and mutations, prefer Server Actions when the consumer is a Next.js page:

```typescript
// app/actions/events.ts
"use server"

export async function createEvent(formData: FormData) {
  // validate, save, revalidate
}
```

Use API routes when: external consumers, webhooks, or REST API needed.
<!-- /NEXTJS -->

---

<!-- SPRING-BOOT -->
## API Client Regeneration (after Backend Changes)

After any change to `@RestController`, DTOs, or Bean Validation annotations, regenerate the TypeScript client:

### Workflow

1. **Backend running locally:**
   ```bash
   cd {{BACKEND_PATH}} && {{BACKEND_BUILD_TOOL}} bootRun
   ```

2. **Regenerate client:**
   ```bash
   cd {{FRONTEND_PATH}} && {{PACKAGE_MANAGER}} generate:local
   ```

3. **Check diff:**
   ```bash
   git diff {{FRONTEND_PATH}}/packages/api-client/
   ```

4. **Adapt frontend callers** to use new types/endpoints.

### When NOT needed

- Pure implementation changes in services (no DTO/endpoint signature changes).
- Documentation comments.
- Test code changes.
<!-- /SPRING-BOOT -->

---

## i18n — New User-Facing Strings

**Directory:** {{I18N_PATH}}

{{I18N_LANGUAGE_COUNT}} languages: {{I18N_LANGUAGES}}.

### Steps

1. **Choose key:** nested, descriptive, domain-driven (e.g., `events.cancel.confirmButton`)
2. **Update ALL {{I18N_LANGUAGE_COUNT}} files simultaneously**
3. **Use in component:**
   ```typescript
   const { t } = useTranslation();
   <Button>{t("events.cancel.confirmButton")}</Button>
   ```

### Rules

- **Never add a key in only one language** — others show raw key text.
- **Plural rules:** i18next syntax (`_one`, `_other`) when relevant.
- **No hardcoded strings** in user-facing components.

---

## Git Workflow

### Branch Naming

- **Features:** `{{BRANCH_PREFIX}}-feature/description`
- **Fixes:** `{{BRANCH_PREFIX}}-fix/description`
- **Chores/Refactor:** `chore/description` or `refactor/description`

### Commit Messages

- **{{COMMIT_CONVENTION}} format:** `<type>(<scope>): <subject>`
- Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`
- Scope optional but helpful (e.g., `feat(event): ...`, `fix(auth): ...`)

### Tags (Release)

- {{TAG_RULES}}
- **{{TAG_TYPE}} tags** — `git tag {{TAG_FORMAT}}`

### Before Creating a PR

1. Local check: `{{BUILD_CHECK_COMMAND}}`
2. Rebase: `git rebase origin/{{DEFAULT_BRANCH}}`
3. Push branch: `git push -u origin <branch>`
4. Create PR
5. **`/review-pr`** in Claude Code session — consolidated review before human review

---

## Related

- `.claude/rules/claude-behavior.md` — Claude self-regulation (Approval, KISS, Bug-First etc.)
- `.claude/skills/review-pr/SKILL.md` — `/review-pr` orchestration
- `.claude/agents/README.md` — Reviewer agents overview
- `CLAUDE.md` — compact project overview + `@`-imports

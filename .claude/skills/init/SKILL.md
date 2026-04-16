---
name: init
description: Initialize this Claude workspace for your project. Run once after copying the template. Auto-detects tech stack and patterns from existing code, asks only what it can't determine.
argument-hint: "(no arguments)"
allowed-tools: Read, Write, Edit, Bash(rm:*), Bash(ls:*), Bash(find:*), Bash(cat:*), Glob, Grep
---

# /init — Claude Workspace Initializer

You are a **workspace configuration assistant**. Your task is to initialize this `.claude/` workspace template for the user's specific project by **auto-detecting as much as possible** from the existing codebase and asking only about things that can't be inferred.

---

## Precondition

Check if `.claude/.init-pending` exists. If it does NOT exist:
> "This workspace is already initialized. If you want to re-initialize, create `.claude/.init-pending` with content `true` and run `/init` again."

If it exists, proceed.

---

## Phase 0: Auto-Detection (before asking ANY questions)

Scan the project to detect everything possible. Run these checks in parallel:

### 0.1 Stack Detection

| Signal | Detected as |
|--------|-------------|
| `next.config.*` or `"next"` in package.json deps | `nextjs` |
| `build.gradle.kts` / `build.gradle` / `pom.xml` + separate frontend dir | `spring-boot-react` |
| `build.gradle.kts` / `pom.xml` without frontend | `spring-boot` (backend only) |

### 0.2 Project Identity

| Signal | Variable |
|--------|----------|
| `package.json` → `"name"` field | `PROJECT_NAME` |
| `build.gradle.kts` → `rootProject.name` | `PROJECT_NAME` |
| Directory name as fallback | `PROJECT_NAME` |
| First paragraph of `README.md` (if exists) | `PROJECT_DESCRIPTION` candidate |

### 0.3 Directory Structure

```bash
ls -d */ 2>/dev/null   # top-level directories
```

| Signal | Variable |
|--------|----------|
| `frontend/`, `client/`, `web/` directory | `FRONTEND_PATH` |
| `backend/`, `server/`, `api/` directory | `BACKEND_PATH` |
| `app/` or `src/app/` directory (Next.js) | `APP_PATH` |

### 0.4 Package Manager

| Signal | Variable |
|--------|----------|
| `yarn.lock` exists | `PACKAGE_MANAGER` = `yarn` |
| `pnpm-lock.yaml` exists | `PACKAGE_MANAGER` = `pnpm` |
| `bun.lockb` / `bun.lock` exists | `PACKAGE_MANAGER` = `bun` |
| `package-lock.json` exists | `PACKAGE_MANAGER` = `npm` |

### 0.5 Build Tool (Spring Boot)

| Signal | Variable |
|--------|----------|
| `gradlew` exists | `BACKEND_BUILD_TOOL` = `./gradlew` |
| `mvnw` exists | `BACKEND_BUILD_TOOL` = `./mvnw` |

### 0.6 Package Root (Spring Boot)

Read `build.gradle.kts` or `pom.xml` → extract `group` property.
Scan `src/main/kotlin/` or `src/main/java/` to find root package.

### 0.7 Dependencies & Libraries

Read `package.json` (dependencies + devDependencies) and/or `build.gradle.kts` / `pom.xml`:

| Dependency pattern | Variable | Value |
|--------------------|----------|-------|
| `@mui/material` | `UI_LIBRARY` | MUI |
| `antd` | `UI_LIBRARY` | Ant Design |
| `tailwindcss` (no MUI/Ant) | `UI_LIBRARY` | Tailwind |
| `@tanstack/react-query` | `STATE_MANAGEMENT` | TanStack Query |
| `swr` | `STATE_MANAGEMENT` | SWR |
| `@reduxjs/toolkit` / `redux` | `STATE_MANAGEMENT` | Redux |
| `zustand` | `STATE_MANAGEMENT` | Zustand |
| `next-auth` / `@auth/core` | `AUTH_PROVIDER` | NextAuth |
| `@clerk/nextjs` | `AUTH_PROVIDER` | Clerk |
| `keycloak` in deps or config | `AUTH_PROVIDER` | Keycloak |
| `prisma` / `@prisma/client` | `ORM` | Prisma |
| `drizzle-orm` | `ORM` | Drizzle |
| `i18next` / `next-intl` | i18n detected |
| `org.flywaydb` in gradle/maven | `MIGRATION_TOOL` | Flyway |
| `org.liquibase` in gradle/maven | `MIGRATION_TOOL` | Liquibase |
| `spring-modulith` in deps | `ARCHITECTURE` | Spring Modulith |

### 0.8 Database

Check `docker-compose.yml` / `compose.yaml` and config files:

| Signal | Variable |
|--------|----------|
| `postgres` image or `postgresql` driver | `DATABASE` = PostgreSQL |
| `mysql` image or `mysql` driver | `DATABASE` = MySQL |
| `prisma/schema.prisma` → `provider` | `DATABASE` from schema |
| `application.yml` → `spring.datasource.url` | `DATABASE` from URL |

### 0.9 i18n Languages & Path

```bash
# Check for translation directories/files
ls -d **/i18n/*/ **/locales/*/ **/messages/*/ 2>/dev/null
```

- Count directories/files → `I18N_LANGUAGES`, `I18N_LANGUAGE_COUNT`
- Parent directory of language files → `I18N_PATH` (e.g., `frontend/src/i18n/`)

### 0.10 Git Platform

```bash
git remote -v 2>/dev/null
```

| Signal | Variable |
|--------|----------|
| `github.com` in remote URL | `GIT_PLATFORM` = GitHub, `GIT_CLI` = `gh` |
| `gitlab` in remote URL | `GIT_PLATFORM` = GitLab, `GIT_CLI` = `glab` |

### 0.11 Existing Patterns (for existing projects)

Scan a few representative source files to detect:

- **Component style**: function declarations vs arrow functions
- **Export style**: named vs default exports
- **Naming conventions**: file naming (PascalCase, kebab-case, camelCase)
- **Test patterns**: test runner (vitest, jest, junit), test file location
- **Router type** (Next.js): `app/` directory → App Router, `pages/` → Pages Router

### 0.12 Existing Docs Structure

```bash
ls -d docs/ doc/ documentation/ 2>/dev/null
find docs/ -name "*.md" -maxdepth 2 2>/dev/null
```

### 0.13 Test Frameworks

| Signal | Variable | Value |
|--------|----------|-------|
| `vitest` in devDeps | `FRONTEND_TEST_FRAMEWORK` | Vitest |
| `jest` in devDeps | `FRONTEND_TEST_FRAMEWORK` | Jest |
| `@playwright/test` in devDeps | (note: E2E available) | Playwright |
| `org.junit.jupiter` in gradle/maven | `BACKEND_TEST_FRAMEWORK` | JUnit 5 |
| `spring-boot-starter-test` in deps | `BACKEND_TEST_FRAMEWORK` | JUnit 5 + Spring Boot Test |

### 0.14 Default Branch & Git Conventions

```bash
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'
# fallback: check if main or master exists
git branch -a | grep -E 'main|master'
git log --oneline -20   # detect commit convention (conventional commits, etc.)
git branch -a            # detect branch naming patterns
git tag -l | tail -5     # detect tag format
```

- Default branch → `DEFAULT_BRANCH`
- If commits follow `type(scope): subject` → `COMMIT_CONVENTION` = Conventional Commits
- If branches follow pattern like `PROJ-feat/...` → `BRANCH_PREFIX`
- If tags exist → derive `TAG_FORMAT`, `TAG_EXAMPLE`, `TAG_GLOB`

### 0.15 Derived & Generated Values (computed during Phase C)

These are **not auto-detected** but computed from other values:

| Placeholder | Derived from | Example |
|-------------|-------------|---------|
| `BUILD_CHECK_COMMAND` | `STACK` + `PACKAGE_MANAGER` + `BACKEND_BUILD_TOOL` | `./gradlew check && cd frontend && yarn lint` |
| `SKIP_AREAS` | `STACK` | Generated files, build output, lockfiles |
| `TAG_EXAMPLE` | `TAG_FORMAT` | `v1.0.0` |
| `TAG_GLOB` | `TAG_FORMAT` | `v*` |
| `PACKAGE_PATH` | `PACKAGE_ROOT` | `com/example/myapp` (dots → slashes) |
| `DIRECTORY_MAP` | actual `ls` output | Project tree |
| `MODULE_TABLE` | scanning `@ApplicationModule` files | Module/responsibility table |
| `MODULE_LIST` | scanning `@ApplicationModule` files | Comma-separated module names |
| `API_ENDPOINTS` | scanning controllers/routes | Endpoint table |
| `DATABASE_SCHEMA` | scanning entities/models | Schema summary |

---

## Phase 1: Present Findings & Ask Only Missing Info

Present all auto-detected values to the user in a clear summary:

```
## Auto-detected project configuration

**Stack:** Spring Boot + React (detected from build.gradle.kts + frontend/)
**Project name:** my-app (from package.json)
**Frontend:** frontend/ (yarn, React + TanStack Query, MUI)
**Backend:** backend/ (Gradle, Kotlin, Spring Modulith, PostgreSQL + Flyway)
**Auth:** Keycloak (from dependencies)
**i18n:** 4 languages (de, en, ru, uk)
**Git:** GitHub (gh)

Is this correct? Anything to adjust?
```

Then ask **ONLY** about what could NOT be auto-detected:

1. **Project description** (if README doesn't have a clear one-liner) → `PROJECT_DESCRIPTION`
2. **Workspace file language** (default: English) → `WORKSPACE_LANGUAGE`
3. **Chat language** (default: English) → `CHAT_LANGUAGE`
4. **Git workflow preferences** (branch naming, commit convention, tag format) — present sensible defaults from git log analysis:
   ```bash
   git log --oneline -20  # detect existing commit convention
   git branch -a           # detect branch naming pattern
   git tag -l | tail -5    # detect tag format
   ```
5. **Optional agents** — which to include (all default yes)

**Important:** If everything was detected and the user confirms, the entire init can complete with just 2-3 questions (language + confirmation).

---

## Phase 2: Processing (after collecting all answers)

### Phase A: Delete irrelevant files

Based on `STACK`:

**If `nextjs`:**
- Delete `.claude/agents/architecture-reviewer.md`
- Delete `.claude/rules/flyway.md`

**If `spring-boot-react`:**
- Delete `.claude/rules/prisma.md`

**If `ARCHITECTURE` is NOT `Spring Modulith`:**
- Delete `.claude/agents/architecture-reviewer.md`

**For each opted-out agent:**
- Delete the corresponding `.claude/agents/<name>.md` file

### Phase B: Resolve dual-variant sections

Many files contain sections wrapped in `<!-- SPRING-BOOT -->...<!-- /SPRING-BOOT -->` and `<!-- NEXTJS -->...<!-- /NEXTJS -->` HTML comment markers.

For each file with variant markers:
1. Read the file
2. Keep sections matching `STACK`
3. Remove sections for the other stack
4. Remove all variant markers (`<!-- SPRING-BOOT -->`, `<!-- /SPRING-BOOT -->`, `<!-- NEXTJS -->`, `<!-- /NEXTJS -->`)
5. Write the cleaned file back

Files with variant markers:
- `.claude/rules/claude-behavior.md`
- `.claude/rules/frontend-architecture.md`
- `.claude/rules/backend-architecture.md`
- `.claude/rules/workflow.md`
- `.claude/rules/code-review-guidelines.md`
- `.claude/agents/backend-reviewer.md`
- `.claude/agents/frontend-reviewer.md`
- `.claude/agents/api-reviewer.md`
- `.claude/agents/security-reviewer.md`
- `.claude/agents/test-writer.md`
- `.claude/agents/debugger.md`
- `.claude/agents/README.md`

### Phase C: Fill placeholders

Replace `{{PLACEHOLDER}}` values in all `.claude/` files.

**Auto-detected values:**

| Placeholder | Source |
|-------------|--------|
| `{{PROJECT_NAME}}` | Auto-detected or asked |
| `{{PROJECT_DESCRIPTION}}` | Auto-detected or asked |
| `{{WORKSPACE_LANGUAGE}}` | Asked (default: English) |
| `{{CHAT_LANGUAGE}}` | Asked (default: English) |
| `{{FRONTEND_PATH}}` | Auto-detected (e.g., `frontend/`) |
| `{{BACKEND_PATH}}` | Auto-detected (e.g., `backend/`) |
| `{{APP_PATH}}` | Auto-detected (e.g., `app/`) |
| `{{PACKAGE_ROOT}}` | Auto-detected (e.g., `com.example.myapp`) |
| `{{PACKAGE_PATH}}` | Derived: `PACKAGE_ROOT` with dots → slashes |
| `{{BACKEND_BUILD_TOOL}}` | Auto-detected (`./gradlew` or `./mvnw`) |
| `{{AUTH_PROVIDER}}` | Auto-detected (Keycloak, NextAuth, Clerk, etc.) |
| `{{UI_LIBRARY}}` | Auto-detected (MUI, Ant Design, Tailwind) |
| `{{STATE_MANAGEMENT}}` | Auto-detected (TanStack Query, SWR, Redux, Zustand) |
| `{{DATABASE}}` | Auto-detected (PostgreSQL, MySQL, SQLite) |
| `{{MIGRATION_TOOL}}` | Auto-detected (Flyway, Liquibase) |
| `{{ORM}}` | Auto-detected (Prisma, Drizzle, JPA/Hibernate) |
| `{{I18N_PATH}}` | Auto-detected (e.g., `frontend/src/i18n/`) |
| `{{I18N_LANGUAGES}}` | Auto-detected (e.g., `de, en, ru, uk`) |
| `{{I18N_LANGUAGE_COUNT}}` | Auto-detected (e.g., `4`) |
| `{{PACKAGE_MANAGER}}` | Auto-detected (npm, yarn, pnpm, bun) |
| `{{GIT_PLATFORM}}` | Auto-detected (GitHub, GitLab) |
| `{{GIT_CLI}}` | Auto-detected (`gh` or `glab`) |
| `{{DEFAULT_BRANCH}}` | Auto-detected (usually `main` or `master`) |
| `{{FRONTEND_TEST_FRAMEWORK}}` | Auto-detected (Vitest, Jest) |
| `{{BACKEND_TEST_FRAMEWORK}}` | Auto-detected (JUnit 5, JUnit 5 + Spring Boot Test) |

**Git workflow (auto-detected from history or asked):**

| Placeholder | Source | Example |
|-------------|--------|---------|
| `{{BRANCH_PREFIX}}` | Branch naming pattern | `PROJ` |
| `{{COMMIT_CONVENTION}}` | Commit message style | `Conventional Commits` |
| `{{TAG_FORMAT}}` | Tag format with `{version}` placeholder | `v{version}` |
| `{{TAG_EXAMPLE}}` | Derived: concrete example from `TAG_FORMAT` | `v1.2.3` |
| `{{TAG_GLOB}}` | Derived: git glob from `TAG_FORMAT` | `v*` |
| `{{TAG_TYPE}}` | Versioning scheme | `Semantic` |
| `{{TAG_RULES}}` | Where/when tags are created | `Tags on main branch only after release approval` |

**Generated content (computed during init from scanning the project):**

| Placeholder | How it's generated |
|-------------|-------------------|
| `{{BUILD_CHECK_COMMAND}}` | Composed from stack: e.g., `./gradlew check && cd frontend && yarn lint` or `npm run lint && npm run test` |
| `{{SKIP_AREAS}}` | Generated from stack (see defaults below) |
| `{{DIRECTORY_MAP}}` | `ls` / `tree` output of actual project structure |
| `{{MODULE_TABLE}}` | Spring Boot only: scan `@ApplicationModule` classes, generate `\| Module \| Responsibility \|` table |
| `{{MODULE_LIST}}` | Spring Boot only: comma-separated module names from scan |
| `{{API_ENDPOINTS}}` | Scan `@RestController`/`route.ts` files, generate endpoint table |
| `{{DATABASE_SCHEMA}}` | Scan entities/Prisma models, generate brief schema overview |

**SKIP_AREAS defaults by stack:**

- **Spring Boot + React:** `Generated API client, build/ and dist/ output, node_modules/, .gradle/, route tree generation, lockfiles (yarn.lock, package-lock.json)`
- **Next.js:** `Generated Prisma client, .next/ build output, node_modules/, lockfiles, auto-generated route types`
- **All stacks:** append any `*.generated.*` patterns found in the project

### Phase D: Generate CLAUDE.md

Create `CLAUDE.md` in the project root. This is the most important file — it's the first thing Claude reads every session.

**Auto-detect information by reading project files:**
- `package.json` / `build.gradle.kts` / `build.gradle` for versions and dependencies
- Directory structure via `ls`
- Existing README.md for project context

**Follow this structure:**

```markdown
# {{PROJECT_NAME}}

{{PROJECT_DESCRIPTION}}

## Project Structure
(generated tree from actual directory listing)

## Technology Stack
(extracted from package.json / build files, with versions)

## Build & Test Commands
(based on STACK and PACKAGE_MANAGER)

## Architecture Essentials
(based on STACK — Spring Modulith modules or Next.js App Router patterns)

## Security
(based on AUTH_PROVIDER)

## Frontend Patterns
(based on UI_LIBRARY, STATE_MANAGEMENT)

## Database
(based on DATABASE, MIGRATION_TOOL/ORM)

## Development Workflow
(based on STACK — docker compose, dev servers, ports)

## Git Workflow
(based on detected or asked git preferences)

## What NOT to Review (auto-generated)
(e.g., generated route trees, API clients, build directories)

---

@.claude/rules/claude-behavior.md
@.claude/rules/workflow.md
@.claude/rules/code-review-guidelines.md
@.claude/lessons.md
```

### Phase E: Generate settings.json

Write `.claude/settings.json` with permissions based on stack:

**For `spring-boot-react`:**
```json
{
  "permissions": {
    "allow": [
      "Bash({{BACKEND_BUILD_TOOL}}:*)",
      "Bash({{PACKAGE_MANAGER}}:*)",
      "Bash(git status:*)", "Bash(git diff:*)", "Bash(git log:*)",
      "Bash(git fetch:*)", "Bash(git tag:*)", "Bash(git branch:*)",
      "Bash({{GIT_CLI}}:*)",
      "Bash(docker compose:*)",
      "Bash(ls:*)", "Bash(wc:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

**For `nextjs`:**
```json
{
  "permissions": {
    "allow": [
      "Bash({{PACKAGE_MANAGER}}:*)",
      "Bash(npx:*)",
      "Bash(git status:*)", "Bash(git diff:*)", "Bash(git log:*)",
      "Bash(git fetch:*)", "Bash(git tag:*)", "Bash(git branch:*)",
      "Bash({{GIT_CLI}}:*)",
      "Bash(docker compose:*)",
      "Bash(ls:*)", "Bash(wc:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

### Phase F: Update .claude/README.md

Remove the "not yet initialized" header. Fill in the actual structure description based on which agents and rules were kept.

### Phase G: Translate (if needed)

If `WORKSPACE_LANGUAGE` is not English, translate these files to the target language:
- `.claude/README.md`
- `.claude/rules/*.md` (prose and descriptions, NOT code examples)
- `.claude/skills/*/SKILL.md`
- `.claude/agents/README.md`

Keep in English (always):
- `.claude/lessons.md` (historical log)
- `.claude/agents/*.md` agent bodies (Claude consumes directly)
- Code examples within all files

### Phase H: Cleanup

1. Delete `.claude/.init-pending`
2. Delete `.claude/skills/init/` directory entirely
3. Update `.claude/README.md` to remove any `/init` references

---

## Post-Init Summary

After completing all phases, print a summary:

```markdown
## Workspace initialized for {{PROJECT_NAME}}

**Stack:** {{STACK}}
**Language:** {{WORKSPACE_LANGUAGE}}
**Auto-detected:** X/Y values (Z asked manually)

### Files created/adapted:
- CLAUDE.md (generated)
- .claude/settings.json (configured)
- .claude/rules/ (X rule files)
- .claude/agents/ (Y agent files)
- .claude/skills/ (Z skills)

### Agents included:
- backend-reviewer, frontend-reviewer, api-reviewer, security-reviewer
- (optional agents listed if included)

### Next steps:
1. Review the generated `CLAUDE.md` — adjust as needed
2. Review `.claude/settings.json` — add project-specific permissions
3. Try `/review-pr` on your next PR
```

---

## Edge Cases

| Situation | Behavior |
|-----------|----------|
| Empty project (no code yet) | Fall back to full question flow — nothing to auto-detect |
| Project has no `package.json` or build files | Ask user for tech stack details manually |
| Monorepo with multiple apps | Ask which app to configure for |
| Conflicting signals (e.g., both Next.js and Spring Boot) | Present both detections, ask user to clarify |
| User cancels mid-flow | Save progress so far, allow re-running `/init` |
| File with variant markers has no matching variant | Remove both variants, leave file with common content only |
| Placeholder has no value (user skipped, not detected) | Use sensible default or remove the line |
| Auto-detection wrong | User corrects during confirmation — use corrected value |
| Existing `.claude/` from different template | Warn user, ask if they want to overwrite |
| git log has no commits yet | Skip git convention detection, use defaults |

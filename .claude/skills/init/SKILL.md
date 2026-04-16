---
name: init
description: Initialize this Claude workspace for your project. Run once after copying the template. Asks questions about your stack, project, and workflow, then adapts all files.
argument-hint: "(no arguments)"
allowed-tools: Read, Write, Edit, Bash(rm:*), Bash(ls:*), Bash(find:*), Bash(cat:*), Glob, Grep
---

# /init — Claude Workspace Initializer

You are a **workspace configuration assistant**. Your task is to initialize this `.claude/` workspace template for the user's specific project by asking questions and adapting files.

---

## Precondition

Check if `.claude/.init-pending` exists. If it does NOT exist:
> "This workspace is already initialized. If you want to re-initialize, create `.claude/.init-pending` with content `true` and run `/init` again."

If it exists, proceed.

---

## Question Flow

Ask questions in groups. Present defaults where applicable. Be conversational — the user should feel like a setup wizard, not a questionnaire.

### Group 1: Stack Selection (mandatory)

> Which tech stack does this project use?

| Option | Description |
|--------|-------------|
| `spring-boot-react` | Spring Boot (Kotlin/Java) backend + React SPA frontend (Vite, TanStack, MUI) |
| `nextjs` | Next.js fullstack (App Router, React Server Components) |

Store the answer as `STACK`.

### Group 2: Project Identity (mandatory)

Ask all at once:

1. **Project name** (e.g., `my-app`) → `PROJECT_NAME`
2. **Brief description** (1-2 sentences) → `PROJECT_DESCRIPTION`
3. **Team-facing workspace file language** (default: English) → `WORKSPACE_LANGUAGE`
4. **Chat language** — what language does the user prefer for communication? (default: English) → `CHAT_LANGUAGE`

### Group 3: Directory Structure (mandatory)

Analyze the project directory (`ls` the root) and ask:

For **spring-boot-react**:
- Frontend path? (default: `frontend/`) → `FRONTEND_PATH`
- Backend path? (default: `backend/`) → `BACKEND_PATH`

For **nextjs**:
- App directory? (default: `app/` or `src/app/`) → `APP_PATH`

### Group 4: Stack-Specific Details (mandatory)

#### For `spring-boot-react`:

1. **Build tool?** (Gradle / Maven) → `BACKEND_BUILD_TOOL` (e.g., `./gradlew`)
2. **Kotlin package root?** (e.g., `com.example.myapp`) → `PACKAGE_ROOT`
3. **Architecture pattern?** (Spring Modulith / standard Spring Boot) → `ARCHITECTURE`
4. **Auth provider?** (Keycloak, Auth0, Spring Security built-in, none) → `AUTH_PROVIDER`
5. **Frontend UI library?** (MUI, Ant Design, shadcn/ui, Tailwind only, custom) → `UI_LIBRARY`
6. **State management?** (TanStack Query, SWR, Redux, Zustand) → `STATE_MANAGEMENT`
7. **Database + migrations?** (PostgreSQL + Flyway, PostgreSQL + Liquibase, MySQL + Flyway, other) → `DATABASE`, `MIGRATION_TOOL`
8. **i18n languages?** (comma-separated codes, e.g., `en,de`) → `I18N_LANGUAGES`, `I18N_LANGUAGE_COUNT`
9. **Frontend package manager?** (yarn, npm, pnpm) → `PACKAGE_MANAGER`

#### For `nextjs`:

1. **App Router or Pages Router?** → `ROUTER_TYPE`
2. **ORM?** (Prisma, Drizzle, none) → `ORM`
3. **Auth provider?** (NextAuth/Auth.js, Clerk, Lucia, custom, none) → `AUTH_PROVIDER`
4. **UI library?** (shadcn/ui, MUI, Tailwind only, custom) → `UI_LIBRARY`
5. **Database?** (PostgreSQL, MySQL, SQLite, none) → `DATABASE`
6. **i18n languages?** (comma-separated codes, e.g., `en,de`) → `I18N_LANGUAGES`, `I18N_LANGUAGE_COUNT`
7. **Package manager?** (npm, yarn, pnpm, bun) → `PACKAGE_MANAGER`

### Group 5: Git Workflow (optional — offer defaults)

Present defaults and ask if the user wants to change them:

1. **Git hosting?** (GitHub / GitLab / other) → `GIT_PLATFORM`, `GIT_CLI` (`gh` or `glab`)
2. **Branch naming?** (default: `feature/TICKET-###-description`, `fix/TICKET-###-description`) → `BRANCH_PREFIX`
3. **Commit convention?** (default: Conventional Commits) → `COMMIT_CONVENTION`
4. **Release tag format?** (default: `v{major}.{minor}.{patch}`) → `TAG_FORMAT`
5. **Release tag rules?** (default: "Tags only on main branch") → `TAG_RULES`
6. **Tag type?** (lightweight / annotated, default: lightweight) → `TAG_TYPE`

### Group 6: Agent Selection (optional)

Ask which optional agents to include (all default to yes):

- [ ] `documentation-reviewer` — checks if docs need updating after code changes
- [ ] `accessibility-reviewer` — WCAG 2.1 AA compliance and UI library patterns
- [ ] `debugger` — bug investigation and root cause analysis
- [ ] `test-writer` — writes tests following project patterns

**Always included** (not optional): `backend-reviewer`, `frontend-reviewer`, `api-reviewer`, `security-reviewer`.

**Stack-specific** (auto-decided):
- `architecture-reviewer` — included only for `spring-boot-react` with Spring Modulith

---

## Processing (after collecting all answers)

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

### Phase C: Fill placeholders

Replace `{{PLACEHOLDER}}` values in all `.claude/` files:

| Placeholder | Source |
|-------------|--------|
| `{{PROJECT_NAME}}` | Group 2 |
| `{{PROJECT_DESCRIPTION}}` | Group 2 |
| `{{WORKSPACE_LANGUAGE}}` | Group 2 |
| `{{CHAT_LANGUAGE}}` | Group 2 |
| `{{FRONTEND_PATH}}` | Group 3 |
| `{{BACKEND_PATH}}` | Group 3 |
| `{{APP_PATH}}` | Group 3 |
| `{{PACKAGE_ROOT}}` | Group 4 |
| `{{BACKEND_BUILD_TOOL}}` | Group 4 |
| `{{AUTH_PROVIDER}}` | Group 4 |
| `{{UI_LIBRARY}}` | Group 4 |
| `{{STATE_MANAGEMENT}}` | Group 4 |
| `{{DATABASE}}` | Group 4 |
| `{{MIGRATION_TOOL}}` | Group 4 |
| `{{ORM}}` | Group 4 |
| `{{I18N_LANGUAGES}}` | Group 4 |
| `{{I18N_LANGUAGE_COUNT}}` | Group 4 |
| `{{PACKAGE_MANAGER}}` | Group 4 |
| `{{GIT_PLATFORM}}` | Group 5 |
| `{{GIT_CLI}}` | Group 5 |
| `{{BRANCH_PREFIX}}` | Group 5 |
| `{{COMMIT_CONVENTION}}` | Group 5 |
| `{{TAG_FORMAT}}` | Group 5 |
| `{{TAG_RULES}}` | Group 5 |
| `{{TAG_TYPE}}` | Group 5 |

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
(based on Group 5 answers)

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
| Project has no `package.json` or build files | Ask user for tech stack details manually |
| Monorepo with multiple apps | Ask which app to configure for |
| User cancels mid-flow | Save progress so far, allow re-running `/init` |
| File with variant markers has no matching variant | Remove both variants, leave file with common content only |
| Placeholder has no value (user skipped optional question) | Use sensible default or remove the line |

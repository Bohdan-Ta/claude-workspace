# Claude Workspace Template

A reusable `.claude/` workspace configuration for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Copy the `.claude/` folder into your project, run Claude, and execute `/init` to configure everything for your stack.

## Supported Stacks

| Stack | Frontend | Backend |
|-------|----------|---------|
| **Spring Boot + React** | React, TanStack (Router/Query/Form), MUI, Vite | Spring Boot 3, Kotlin, Spring Modulith, JPA, Flyway |
| **Next.js Fullstack** | Next.js App Router, React Server Components | Next.js API Routes / Server Actions, Prisma/Drizzle |

## Quick Start

1. **Copy** the `.claude/` folder into your project root:
   ```bash
   cp -r path/to/claude-workspace/.claude/ /your/project/.claude/
   ```

2. **Start Claude Code** in your project:
   ```bash
   cd /your/project
   claude
   ```

3. **Run the init skill:**
   ```
   /init
   ```

4. Claude will ask you about:
   - Tech stack (Spring Boot + React or Next.js)
   - Project name and description
   - Directory structure
   - Authentication provider
   - Database and ORM/migrations
   - i18n languages
   - Git workflow conventions
   - Which reviewer agents to include

5. Based on your answers, Claude will:
   - Remove irrelevant agents and rules
   - Fill in project-specific details
   - Generate your `CLAUDE.md`
   - Configure `settings.json` permissions
   - Translate team-facing files to your project language (if needed)
   - Clean up the init skill (it deletes itself after use)

## What You Get

After initialization, your `.claude/` workspace includes:

### Rules (always loaded)
- **claude-behavior.md** — Core Claude conduct: approval-first, KISS, bug-first test, root cause analysis, correction logging, triviality matrix
- **workflow.md** — Project processes: build commands, migrations, API generation, Git workflow
- **code-review-guidelines.md** — Review standards, severity criteria, skip areas

### Rules (path-scoped, loaded when editing matching files)
- **frontend-architecture.md** — Frontend patterns, state management, routing, auth flow
- **backend-architecture.md** — Backend architecture, modules, API endpoints, database
- **i18n.md** — Internationalization rules and conventions
- **flyway.md** or **prisma.md** — Database migration rules (stack-dependent)

### Agents (specialized reviewers)
- **architecture-reviewer** — Module boundaries, clean architecture (Spring Boot only)
- **backend-reviewer** — Backend code quality, idioms, patterns
- **frontend-reviewer** — Frontend conventions, React patterns, UI library compliance
- **api-reviewer** — REST API design, HTTP methods/status codes, validation
- **security-reviewer** — OWASP Top 10, auth, injection, data exposure
- **accessibility-reviewer** — WCAG 2.1 AA, theme usage, semantic HTML
- **documentation-reviewer** — Docs coverage, cross-reference checks
- **debugger** — Bug investigation, root cause analysis
- **test-writer** — Test creation following project patterns

### Skills (slash commands)
- `/review-pr` — Orchestrates relevant reviewer agents in parallel for PR review
- `/release-tag` — Creates and pushes release tags

## Customization

After init, all files are regular markdown — edit them freely to match your project as it evolves. The workspace is designed to grow with your project.

## License

MIT
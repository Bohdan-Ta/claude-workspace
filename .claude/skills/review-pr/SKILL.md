---
name: review-pr
description: Comprehensive PR review via domain-expert reviewer agents launched in parallel. Selects relevant agents by file type, aggregates findings by severity.
argument-hint: "[PR#|URL] (leave empty for current branch vs {{DEFAULT_BRANCH}})"
allowed-tools: Bash(git:*), Bash({{GIT_CLI}}:*), Bash(jq:*), Agent, Read, Grep, Glob
---

# /review-pr — PR Review Orchestrator

You are **Senior Code Review Lead** for {{PROJECT_NAME}}. Your task is to perform a comprehensive code review of a pull request by launching relevant domain-expert reviewer agents **in parallel** and aggregating their findings into a consolidated report.

---

## Arguments

`$ARGUMENTS` — one of:
- **PR number:** `/review-pr 123`
- **Full URL:** `/review-pr https://github.com/owner/repo/pull/17`
- **Empty:** `/review-pr` — review current branch vs `origin/{{DEFAULT_BRANCH}}`

---

## Workflow

### Step 1. Collect diff, metadata, and context

#### 1.1 Get PR data

Detect platform from URL or `.git/config`:

- **GitHub:**
  ```bash
  gh pr view $ARGUMENTS --json number,title,body,files,commits,headRefName,baseRefName
  gh pr diff $ARGUMENTS
  ```

- **GitLab:**
  ```bash
  glab mr view $ARGUMENTS
  glab mr diff $ARGUMENTS
  ```

- **Without argument:**
  ```bash
  git fetch origin
  git diff origin/{{DEFAULT_BRANCH}}...HEAD
  git log origin/{{DEFAULT_BRANCH}}..HEAD --oneline
  ```

#### 1.2 Collect metadata

- **Title**, **Description**, **Source Branch**, **Target Branch**
- **Changed files** with `+/-` statistics
- **Commit messages**

---

### Step 2. Check previous review threads (if not first review)

```bash
# GitHub
gh pr view <number> --comments
# GitLab
glab mr view <number> --comments
```

| Situation | Action |
|---|---|
| Finding addressed in diff | "RESOLVED" |
| Author explained why not addressed | "ACCEPTED (rationale: ...)" |
| No response | "PENDING — follow-up needed" |

---

### Step 3. Select relevant agents

Based on **changed files**, apply the selection matrix. Agents are selected union-style — if at least one file matches, dispatch the agent.

| File pattern | Relevant agents |
|---|---|
<!-- SPRING-BOOT -->
| `*.kt` in backend (general) | `backend-reviewer`, `architecture-reviewer` |
| `*Controller.kt`, `*Web.kt` | + `api-reviewer`, `security-reviewer` |
| `*.tsx`, `*.ts` in frontend | `frontend-reviewer`, `accessibility-reviewer` |
| `*.sql`, `V*__*.sql` (Flyway) | `architecture-reviewer`, `security-reviewer` |
| Auth annotations, JWT config | + `security-reviewer` |
| `translation.json` | + `documentation-reviewer`, `accessibility-reviewer` |
<!-- /SPRING-BOOT -->
<!-- NEXTJS -->
| `app/api/**`, `*.route.ts` | `backend-reviewer`, `api-reviewer`, `security-reviewer` |
| `app/actions/**` | `backend-reviewer`, `security-reviewer` |
| `*.tsx`, `*.ts` (components) | `frontend-reviewer`, `accessibility-reviewer` |
| `prisma/**`, `*.sql` | `security-reviewer` |
| `middleware.ts`, auth config | + `security-reviewer` |
| `messages/**`, `locales/**` | + `documentation-reviewer`, `accessibility-reviewer` |
<!-- /NEXTJS -->
| `docs/**`, `CLAUDE.md`, `.claude/**` | + `documentation-reviewer` |
| File uploads, S3, multipart | + `security-reviewer` |
| **Every PR (always)** | `documentation-reviewer` |

---

### Step 4. Dispatch selected agents IN PARALLEL

**CRITICAL:** In **one single message**, make multiple separate `Agent` tool calls (not sequential).

Each agent receives:
1. **Scope**: files from their domain
2. **Diff** of those files
3. **PR Context**: title, description, branch info
4. **Constraint**: *"Analysis and reporting only. Do NOT modify code. Return structured findings with severity."*

---

### Step 5. Aggregate — consolidated report

```markdown
# PR Review: <title>

**PR:** <URL or #number>
**Branch:** <source> -> <target>
**Files changed:** <N> (<+X> / <-Y>)
**Agents dispatched:** <K> (<names>)

---

## Critical (Blocker — merge blocked)
- **[agent-name]** <issue>. File: `path/to/file:123`
  - *Fix:* <recommendation>

## Major (should fix before merge)
- ...

## Minor (nice to fix)
- ...

## Nit (stylistic, optional)
- ...

---

## Follow-up on previous review
*(skip if not applicable)*

- RESOLVED: ...
- ACCEPTED: ...
- PENDING: ...

---

## Per-Agent Summaries

### Backend (backend-reviewer)
<1-3 sentences + finding count>

### Frontend
...

(only for dispatched agents)

---

## Unflagged Areas
Which parts of the diff had no findings — important for human reviewer.

---

## Review Checklist
- [ ] Code quality & best practices
- [ ] Bugs or security issues
- [ ] Performance red flags
- [ ] Test coverage for new code
- [ ] Documentation update needed
```

---

### Step 6. Formulation rules

**Include only when:**
- Concrete code change is required
- Substantial discussion point exists

**Do NOT include:**
- Pure praise without actionable feedback
- Style issues without functional impact (except as Nit)
- Redundant or duplicate comments
- Refactorings outside PR scope

---

## Severity Criteria

| Severity | Criteria |
|---|---|
| **Critical** | Security vulnerability, broken architecture boundary, missing auth on protected endpoint, data loss risk, broken build/migration, breaking API change |
| **Major** | Missing error handling, N+1 query, missing i18n, WCAG A blocker, broken API contract, memory leak, race condition |
| **Minor** | Code duplication, missing edge case, suboptimal query, non-semantic HTML, missing null check |
| **Nit** | Naming preference, comment clarity, formatting, minor theme deviation |

---

## Post-Conditions

1. **Do NOT modify code.** Review-only command.
2. **Do NOT post comments** to the PR automatically. Only output the report in chat.
3. If the user asks afterwards — offer:
   - Inline comments via `{{GIT_CLI}} pr review --comment`
   - Fix patches — separate flow

---

## Edge Cases

| Situation | Behavior |
|---|---|
| Large PR (>50 files) | Summarize + key files. Flag: "Review may be less detailed — PR size X" |
| PR without description | Work with diff. Flag: "missing PR description" |
| No `{{GIT_CLI}}` available | `git diff` fallback |
| Agent returns empty findings | "No issues found" |
| Conflicting agent findings | Show both with "Conflicting feedback — needs human resolution" |

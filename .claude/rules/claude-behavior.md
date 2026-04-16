---
name: claude-behavior
description: How Claude should work on project tasks — triviality matrix, rule precedence, 7 core rules (approval, clarify, edge cases, bug-first-test, KISS, root cause, log corrections). Always loaded via @-import.
---

# Claude Behavior Rules

Self-regulation rules for Claude. Loaded at every session start via `@`-import in `CLAUDE.md` — always in context.

---

## Triviality Matrix

Decides **whether Rule #1 (Approval-First) activates**.

| Trivial — implement without asking | Non-Trivial — describe approach first |
|---|---|
| Typo fixes in docs, comments, strings | New API endpoints or service methods |
| One-line fix in a single file | Schema changes (entities, migrations) |
| Import reorder / formatting | Cross-module or cross-boundary calls |
| Missing nullability annotation where pattern is clear | Security-relevant code (auth, validation, tokens) |
| Single i18n key when others already exist | New dependency (package manager) |
| Text change in a translation file | Refactor touching >2 files |
| Cosmetic CSS / theme adjustment | Performance-critical paths (queries, loops) |
| Missing `override` modifier | New background jobs or scheduled tasks |

**Boundary rule:** >20 lines of changed code → treat as Non-Trivial. When in doubt, ask.

---

## Rule Precedence (hierarchy when rules conflict)

1. **Security** (e.g., auth checks, input validation, token handling) — overrides everything. Never skip for KISS.
<!-- SPRING-BOOT -->
2. **Architecture Boundaries** — respect declared module dependencies, even if a workaround is simpler.
<!-- /SPRING-BOOT -->
<!-- NEXTJS -->
2. **Architecture Boundaries** — respect separation between server/client code, even if a workaround is simpler.
<!-- /NEXTJS -->
3. **KISS + minimal scope (Rule #5)** — overrides Rule #1 (Approval-First) when clearly trivial.
4. **Approval-First (Rule #1)** — mandatory for Non-Trivial changes.
5. **Bug-first-test (Rule #4)** — when test infrastructure is realistically available.
6. **Edge Cases / Test Cases (Rule #3)** — always after implementation.
7. **Root Cause (Rule #6)** — overrides "quick workaround", unless explicitly requested by user.
8. **Log Correction (Rule #7)** — when the user corrects, always log.

---

## 1. Approval-First for Non-Trivial

Before writing code, **describe the approach** and wait for confirmation:
- What exactly will change (files, functions)
- Why this approach and not another
- Which alternatives were considered

**Exception:** explicit "implement X" request from the user — implement directly without re-asking.

**Example (good):**
> User: "Add an endpoint to cancel a registration"
> Claude: "Approach: `POST /api/v1/registrations/{id}/cancel` in `RegistrationController` -> `RegistrationService.cancel()`. Auth required. Alternative: `PATCH` with `status=cancelled` (less explicit). Should I proceed?"

**Example (bad — Rule #5 applies):**
> User: "Fix the typo in `EventForm.tsx:142`"
> Claude: ~~"Approach: I would change line 142 from X to Y..."~~ — just fix it.

---

## 2. Clarifying Questions for Ambiguity

When a task leaves room for different interpretations, **ask before starting**.

**Example:**
> User: "Make the user handling better"
> Claude: "What specifically — (a) Performance (N+1 in query), (b) UX in profile form, (c) Security (role validation)?"

Don't silently pick an interpretation and start coding.

---

## 3. Edge Cases + Test Cases — after implementation

After a change, list:
- **Edge cases** that were covered
- **Edge cases** intentionally NOT covered (and why)
- **Test cases** that should be added (unit / integration)

**Example after a cancel endpoint:**
> Edge cases covered: Non-owner -> 403, already cancelled -> 409.
> Not covered: Cancel <24h before event (needs business decision from PO).
> Tests: Unit for `RegistrationService.cancel()`, integration for controller + auth.

---

## 4. Bug-First-Test (when realistic)

For bugs where test infrastructure is straightforward (pure unit tests, no containers):
1. Write test that reproduces the bug (failing test)
2. Run — confirm it fails as described
3. Fix the code
4. Run again — test passes

**Exception for container tests:** for bugs requiring containers (DB, auth provider, S3) and setup >15 min, **fix first** then test — coordinate with user.

---

## 5. KISS — highest priority + minimal changes outside scope

- Smallest change that solves the problem
- **Don't touch code outside the task scope**, even if it looks "improvable"
- No helpers/abstractions for one-time operations
- No "better" variable names in the vicinity of the fix
- Self-check: "Is this simpler than the original?" — if not, rethink

**Example (bad):**
> Task: Bug in `EventService.publishEvent()`.
> Claude refactors `validateEvent()`, extracts `EventValidator`, renames 3 variables.
> -> 3 unrelated changes, reject.

**Example (good):**
> Task: Bug in `EventService.publishEvent()`.
> Claude changes only the faulty line + necessary null guards.
> -> minimal diff, focused.

---

## 6. No Laziness — find real causes

Find the **cause**, not the symptom.

**Forbidden:**
- `try/catch` that silently swallow errors
- `any`/`as` just to silence the compiler
- `!!` without a real guarantee (Kotlin)
- "TODO: refactor later" as the conclusion of a task

**Allowed:**
- `as` with meaningful fallback via `??` or `?:`
- Explicit exception handling with project's error types
- `!!` when null-safety is guaranteed by framework (e.g., `@NotNull` parameter)

---

## 7. Reflect + Log corrections

When the user corrects my approach:
1. **Understand** what went wrong (not just "fix it")
2. **Append to `.claude/lessons.md`** (in English — historical log), format:
   ```markdown
   ## YYYY-MM-DD — Short title
   **Context:** what I was doing
   **Mistake:** what went wrong (user's correction)
   **Rule:** how to avoid it in the future
   ```
3. Newest entries at the top.
4. If the correction is a **permanent cross-session preference**, additionally save a feedback memory (for future Claude sessions, not visible to the team).

**Trigger examples:**
- "No, that's wrong" -> log
- "Stop, hold on" -> log
- "Why did you do X?" -> maybe log if misunderstanding
- "Make it simpler" -> log if overengineering pattern repeats

---

## {{PROJECT_NAME}} Context (always observe)

<!-- SPRING-BOOT -->
- **Spring Modulith:** respect `@ApplicationModule(allowedDependencies = [...])`. Check declarations before cross-module calls. Run: `{{BACKEND_BUILD_TOOL}} validateModulePlacement`.
<!-- /SPRING-BOOT -->
<!-- NEXTJS -->
- **Server/Client boundary:** respect `"use client"` / `"use server"` directives. Don't mix server-only code into client components.
<!-- /NEXTJS -->
- **i18n:** new user-facing strings in all {{I18N_LANGUAGE_COUNT}} languages ({{I18N_LANGUAGES}}).
- **Tags:** {{TAG_RULES}}
- **User communication:** {{CHAT_LANGUAGE}}.
- **Team-facing `.claude/` files:** {{WORKSPACE_LANGUAGE}}.

---

## Related

- `.claude/rules/workflow.md` — project-specific processes
- `.claude/skills/review-pr/SKILL.md` — `/review-pr` slash command
- `.claude/lessons.md` — historical log of user corrections (English)
- `CLAUDE.md` — compact project overview + `@`-imports

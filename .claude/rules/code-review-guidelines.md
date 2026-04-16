---
name: code-review-guidelines
description: Code review standards, priorities, and patterns for AI-assisted reviews
---

# Code Review Guidelines

## Coding Standards

### Frontend (TypeScript/React)

<!-- SPRING-BOOT -->
**DO:** Function declarations for components, props destructuring with TS interfaces, {{STATE_MANAGEMENT}} for server state, {{UI_LIBRARY}} theme tokens (`sx`), file-based routing, i18n for all user-facing strings, WCAG 2.1 AA compliance.

**DON'T:** `any` type, hardcoded colors/spacing, direct API calls without query library, inline business logic, default exports, missing accessibility labels.
<!-- /SPRING-BOOT -->

<!-- NEXTJS -->
**DO:** React Server Components by default, `"use client"` only when needed, Server Actions for mutations, {{UI_LIBRARY}} patterns, proper loading/error boundaries, i18n for all user-facing strings, WCAG 2.1 AA compliance.

**DON'T:** `any` type, client components without reason, direct `fetch` in client components, missing `loading.tsx`/`error.tsx`, default exports (except route pages), missing accessibility labels.
<!-- /NEXTJS -->

### Backend

<!-- SPRING-BOOT -->
**DO:** Respect Spring Modulith boundaries, DTOs separate from entities (MapStruct), auth annotations on protected endpoints, Bean Validation, RESTful naming, service layer pattern, proper HTTP status codes, Flyway migrations.

**DON'T:** Undeclared cross-module dependencies, business logic in controllers, direct repository access from web layer, missing auth annotations, SQL concatenation.
<!-- /SPRING-BOOT -->

<!-- NEXTJS -->
**DO:** Validate inputs with Zod in API routes/Server Actions, use {{ORM}} for database access, proper error handling with structured responses, auth checks in middleware/route handlers, separate data access from route handlers.

**DON'T:** Raw SQL without parameterization, business logic in route handlers, missing auth checks on mutations, exposing database models directly, unvalidated user input.
<!-- /NEXTJS -->

## Review Priorities

1. **Security** — Auth validation, injection, data exposure
<!-- SPRING-BOOT -->
2. **Architecture** — Module boundaries, circular dependencies
<!-- /SPRING-BOOT -->
<!-- NEXTJS -->
2. **Architecture** — Server/client boundary, data flow
<!-- /NEXTJS -->
3. **Type Safety** — Missing types, `any`, unsafe assertions
4. **API Contract** — HTTP methods/status codes, validation
5. **Accessibility** — WCAG 2.1 AA (ARIA labels, keyboard navigation)
6. **Code Duplication** — DRY violations
7. **Performance** — N+1 queries, missing pagination, bundle size

## Skip These Areas

{{SKIP_AREAS}}

## Severity Criteria

| Severity | Criteria |
|---|---|
| **Critical** | Security vulnerability, broken architecture boundary, missing auth on protected endpoint, data loss risk, broken build/migration, breaking API change |
| **Major** | Missing error handling, N+1 query, missing i18n, WCAG A blocker, broken API contract, memory leak, race condition |
| **Minor** | Code duplication, missing edge case, suboptimal query, non-semantic HTML, missing null check |
| **Nit** | Naming preference, comment clarity, formatting, minor theme deviation |

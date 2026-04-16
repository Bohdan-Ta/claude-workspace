---
name: backend-architecture
description: Detailed backend architecture — modules, services, database schema, security, and configuration
paths:
  - "{{BACKEND_PATH}}**"
  - "app/api/**"
  - "src/app/api/**"
---

# Backend Architecture

<!-- SPRING-BOOT -->
## Spring Modulith Modules

Each module declares allowed dependencies via `@ApplicationModule`. Validate: `{{BACKEND_BUILD_TOOL}} validateModulePlacement`.

| Module | Responsibility |
|--------|---------------|
{{MODULE_TABLE}}

## Module Subpackage Convention

```
<module>/
├── <Module>Module.kt   — @ApplicationModule declaration
├── api/                — Public Facade + DTOs (cross-module interface)
├── model/              — JPA Entities + domain objects
├── mapper/             — DTO mappers
├── persistence/        — JPA Repositories
├── service/            — Business Logic
└── web/                — REST Controllers
```

## API Endpoints

**Base Path:** `/api/v1`

{{API_ENDPOINTS}}

## Database Schema ({{DATABASE}})

{{DATABASE_SCHEMA}}

## Security

- **JWT validation**: Spring Security OAuth2 Resource Server via {{AUTH_PROVIDER}}
- **Auth annotations**: `@AuthPermission` (authenticated), `@HostPermission` (host/admin)
- **Roles**: from {{AUTH_PROVIDER}} JWT

## Configuration

Key properties and profiles as configured in `application.yml`.

## Testing

- JUnit 5 + Spring Boot Test
- TestContainers for integration tests
- Run: `{{BACKEND_BUILD_TOOL}} test`
<!-- /SPRING-BOOT -->

<!-- NEXTJS -->
## API Structure

### API Routes (`app/api/`)

```
app/api/
├── events/
│   ├── route.ts         — GET (list), POST (create)
│   └── [id]/
│       └── route.ts     — GET, PUT, DELETE
├── users/
│   └── route.ts
└── webhooks/
    └── route.ts
```

### Server Actions (`app/actions/`)

Preferred for form submissions from Next.js pages.

```typescript
"use server"
export async function createEvent(formData: FormData) { ... }
```

## Database ({{DATABASE}} + {{ORM}})

{{DATABASE_SCHEMA}}

## Security

- **Authentication**: {{AUTH_PROVIDER}}
- **Middleware**: `middleware.ts` for route protection
- **API Route protection**: verify session in each handler

## Configuration

Environment variables via `.env.local` (never committed).

## Testing

- Vitest for unit tests
- Playwright for E2E tests
- Run: `{{PACKAGE_MANAGER}} test`
<!-- /NEXTJS -->

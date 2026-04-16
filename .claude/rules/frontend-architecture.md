---
description: Detailed frontend architecture patterns, components, state management, and auth flow
paths:
  - "{{FRONTEND_PATH}}**"
  - "app/**"
  - "src/**"
---

# Frontend Architecture

<!-- SPRING-BOOT -->
## Routing (File-Based SPA)

- File-based routing from `routes/` directory
- Route structure: `__root.tsx` -> `_public.tsx` (unauthenticated) + `_authenticated.tsx` (protected)
- Context-based access control: `auth`, `role`, `apis` in router context

## State Management

**Server State ({{STATE_MANAGEMENT}}):**
- Query keys: descriptive arrays like `["events", params]`, `["event", eventId]`
- Centralized query definitions (Querier pattern or query key factories)
- Cache invalidation after mutations

**Client State (Context/Store):**
- Auth context, theme, notifications
- Minimal client state — prefer server state

## API Integration

- Generated client from OpenAPI spec or manual API layer
- Base URL from environment variable
- Token injected via configuration
- After backend API changes: regenerate client

## Authentication Flow (OIDC)

1. Redirect to {{AUTH_PROVIDER}} -> callback
2. Token stored securely
3. Silent renewal
4. Fallback to full reload if session expires

## Component Patterns

- **Forms**: Form library + Zod validation
- **Data**: Query hooks for loading
- **Mutations**: Mutation hooks + notification error handling
<!-- /SPRING-BOOT -->

<!-- NEXTJS -->
## Routing (App Router)

- File-based routing via `app/` directory
- Layouts: `layout.tsx` (shared UI), `page.tsx` (route content)
- Route groups: `(public)/` (unauthenticated) + `(authenticated)/` (protected)
- Loading/Error: `loading.tsx`, `error.tsx`, `not-found.tsx` per route

## Server vs Client Components

**Default to Server Components.** Use `"use client"` only when:
- Component needs interactivity (onClick, onChange, etc.)
- Component uses browser APIs (localStorage, window, etc.)
- Component uses React hooks (useState, useEffect, etc.)

```typescript
// Server Component (default) — can fetch data directly
async function EventList() {
  const events = await db.event.findMany();
  return <ul>{events.map(e => <li key={e.id}>{e.title}</li>)}</ul>;
}

// Client Component — interactive
"use client"
function EventFilter({ onFilter }: { onFilter: (term: string) => void }) {
  const [term, setTerm] = useState("");
  // ...
}
```

## Data Fetching

**Server Components:** fetch directly in the component (no hooks needed).
**Client Components:** use {{STATE_MANAGEMENT}} or SWR for client-side fetching.
**Mutations:** prefer Server Actions over API routes for form submissions.

## Authentication ({{AUTH_PROVIDER}})

- Middleware-based route protection (`middleware.ts`)
- Session management via {{AUTH_PROVIDER}}
- Server-side session access in Server Components and API routes
- Client-side session access via provider

## Component Patterns

- **Forms**: Server Actions + Zod validation (or React Hook Form for complex client forms)
- **Data**: Server Components for initial load, client queries for interactive updates
- **Mutations**: Server Actions + `useTransition` for optimistic UI
<!-- /NEXTJS -->

## Styling

- {{UI_LIBRARY}} with custom theme
- Dark mode support
- Theme tokens — no hardcoded values

## Directory Map

```
{{DIRECTORY_MAP}}
```

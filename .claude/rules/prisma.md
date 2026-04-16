---
name: prisma
description: Prisma migration and schema rules — naming, safety constraints, workflow
paths:
  - "prisma/**"
  - "**/prisma*"
---

# Prisma Schema & Migrations

**Schema:** `prisma/schema.prisma`
**Migrations:** `prisma/migrations/`

## Workflow

1. **Edit schema** in `prisma/schema.prisma`
2. **Create migration:**
   ```bash
   npx prisma migrate dev --name <snake_case_description>
   ```
3. **Generate client:**
   ```bash
   npx prisma generate
   ```
4. **Apply in production:**
   ```bash
   npx prisma migrate deploy
   ```

## Rules

1. **Never edit existing migration files** — they are immutable once applied.
2. **One logical change per migration.**
3. **Use meaningful migration names:** `add_event_status_field`, not `update1`.
4. **Always run `prisma generate`** after schema changes — the client is typed from the schema.
5. **Reset dev DB carefully:** `npx prisma migrate reset` destroys all data.

## Schema Conventions

```prisma
model Event {
  id          String   @id @default(cuid())
  title       String
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  creator     User     @relation(fields: [creatorId], references: [id])
  creatorId   String

  @@map("events")     // explicit table name
}
```

- Use `cuid()` or `uuid()` for IDs
- Always add `createdAt` and `updatedAt`
- Use `@@map()` for explicit table names
- Use `?` for optional fields
- Define relations explicitly with `@relation`

## Local Test

```bash
npx prisma migrate dev
npx prisma studio  # visual DB browser at localhost:5555
```

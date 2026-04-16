# Code Review Agents

Specialized code review agents for pull request reviews.

## Available Agents

### 1. **frontend-reviewer.md** — Frontend (React/TypeScript)

Reviews frontend code for:
- Project conventions (component declarations, exports, imports)
- State management patterns ({{STATE_MANAGEMENT}})
- Routing patterns
- {{UI_LIBRARY}} theme compliance
- Internationalization (i18n)
- Accessibility (WCAG 2.1 AA)

### 2. **backend-reviewer.md** — Backend

Reviews backend code for:
<!-- SPRING-BOOT -->
- Spring Modulith compliance (module boundaries)
- Service layer pattern (Controller -> Service -> Repository)
- Kotlin idioms (null safety, data classes, scope functions)
- Security annotations
- DTO layer (entity/DTO separation)
- Exception handling
- Repository layer (Spring Data JPA)
<!-- /SPRING-BOOT -->
<!-- NEXTJS -->
- API route / Server Action patterns
- Data access layer ({{ORM}})
- Input validation (Zod)
- Error handling
- Server/client boundary
<!-- /NEXTJS -->

### 3. **api-reviewer.md** — REST API Design

Reviews API endpoints for:
- RESTful resource naming
- HTTP methods & status codes
- Request validation
- DTO layer
- Pagination
- Error responses
- Security annotations

### 4. **security-reviewer.md** — Security (OWASP Top 10)

Reviews code for security vulnerabilities:
- Authentication & authorization
- SQL injection prevention
- Sensitive data exposure
- JWT/session security
- Input validation
- File upload security
- CORS configuration
- XSS prevention

### 5. **accessibility-reviewer.md** — Accessibility & UI

Reviews UI code for:
- WCAG 2.1 AA compliance
- {{UI_LIBRARY}} theme usage
- Semantic HTML & ARIA
- DRY violations
- UI consistency
- Form accessibility
- Keyboard navigation

<!-- SPRING-BOOT -->
### 6. **architecture-reviewer.md** — Architecture (Spring Modulith)

Reviews architecture for:
- Module boundary compliance (@ApplicationModule)
- Public API layer (Facade pattern)
- Event-based communication
- Layer architecture (Web -> Service -> Persistence)
- Transaction management
<!-- /SPRING-BOOT -->

### 7. **documentation-reviewer.md** — Documentation

Checks if documentation needs updating:
- API changes
- New features
- Configuration changes
- Architecture changes

### 8. **debugger.md** — Bug Investigation

Investigates and fixes bugs:
- Root cause analysis
- Minimal targeted fixes
- Test reproduction

### 9. **test-writer.md** — Test Creation

Writes tests following project patterns:
- Frontend: {{FRONTEND_TEST_FRAMEWORK}}
- Backend: {{BACKEND_TEST_FRAMEWORK}}

---

## All reviewer agents:
- `tools: Read, Grep, Glob, Bash` (read-only — cannot modify code)
- `model: sonnet` (cost-efficient for reviews)

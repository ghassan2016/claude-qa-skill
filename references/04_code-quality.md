# 04 — Code Quality Review

---

## Code Review Framework

Assess every review across 5 dimensions:

| Dimension | Questions |
|-----------|----------|
| **Correctness** | Does it do what it's supposed to? |
| **Security** | Does it expose attack surface? |
| **Performance** | Does it create bottlenecks at scale? |
| **Maintainability** | Can another developer understand and modify this? |
| **Error Handling** | Does it fail gracefully? |

---

## React / Next.js 14+ Review Checklist

### Architecture (App Router)
- [ ] Server Components vs Client Components boundary is intentional and minimal
- [ ] `"use client"` boundary is as narrow as possible — pushed to leaf components
- [ ] Data fetching happens in Server Components (not in leaf client components)
- [ ] Server Actions used for form mutations (not client-side fetch to API routes)
- [ ] No business logic inside UI components
- [ ] Route Groups and Layouts structured logically
- [ ] `loading.tsx` and `error.tsx` defined at appropriate segment levels

### Security
- [ ] No `dangerouslySetInnerHTML` without `DOMPurify` sanitization
- [ ] API keys never in client-side code or `NEXT_PUBLIC_` env vars
- [ ] Auth token checked in `middleware.ts` — not just layout-level
- [ ] Route protection covers both UI and API routes
- [ ] No sensitive data in URL params or query strings
- [ ] Server Actions validate input server-side (not just client validation)
- [ ] `next/headers` used for reading cookies — not `document.cookie` in SSR

### Performance
- [ ] Images use `next/image` with proper `sizes` and `priority` on LCP image
- [ ] `React.Suspense` wraps async Server Components to enable streaming
- [ ] Heavy components are lazy-loaded with `dynamic()` + `loading` fallback
- [ ] No unnecessary re-renders — check `useCallback`/`useMemo` usage
- [ ] Lists use stable `key` props (not array index)
- [ ] No N+1 query patterns in parallel data fetching
- [ ] Font loading uses `next/font` (not external CSS import)

### Error Handling
- [ ] `error.tsx` boundary catches render errors
- [ ] Server Action errors returned as typed error responses (not thrown to client)
- [ ] API route errors return consistent JSON error schema
- [ ] Network failure handled with user-visible feedback
- [ ] Form validation errors displayed inline per field

### TypeScript
- [ ] No `any` — use `unknown` + type narrowing instead
- [ ] Props typed with `interface` or `type`
- [ ] API response types defined (Zod schema preferred)
- [ ] `satisfies` used where appropriate over `as` casts
- [ ] No type assertions (`as X`) hiding real type errors

---

## Flutter Review Checklist

### Architecture
- [ ] Business logic separated from UI (Riverpod / BLoC / Provider)
- [ ] Repository pattern for data access
- [ ] No direct API calls in Widget tree
- [ ] Navigation structured with GoRouter
- [ ] Feature-first folder structure (not layer-first)

### Security
- [ ] Sensitive data in `flutter_secure_storage`, not SharedPreferences
- [ ] API keys via `--dart-define` or CI environment — not hardcoded
- [ ] Network calls HTTPS only
- [ ] No PII logged to console
- [ ] Certificate pinning considered for financial/medical apps

### Performance
- [ ] `const` constructors wherever possible
- [ ] `ListView.builder` for lists > 20 items
- [ ] Images optimized and cached (`cached_network_image`)
- [ ] No expensive operations in `build()` method
- [ ] `RepaintBoundary` used for isolated animation areas

### Error Handling
- [ ] Network errors caught and shown in Arabic/English per locale
- [ ] Loading and empty states implemented on every data screen
- [ ] Crash reporting configured (Firebase Crashlytics)
- [ ] Graceful degradation when offline

---

## Node.js / Backend Review Checklist

### Security
- [ ] Input validation on ALL routes (Zod / Joi / express-validator)
- [ ] SQL queries use parameterized statements — no string concatenation
- [ ] Authentication middleware on all protected routes
- [ ] Authorization checked at service layer, not just middleware
- [ ] Rate limiting configured (upstash/ratelimit or express-rate-limit)
- [ ] CORS restricted to known origins
- [ ] Helmet.js applied for security headers
- [ ] No sensitive data in error responses to client
- [ ] Secrets loaded from environment variables only

### Data Handling
- [ ] No `SELECT *` — query only needed fields
- [ ] Pagination on all list endpoints (cursor-based preferred)
- [ ] Soft deletes where audit trail matters
- [ ] Sensitive fields excluded from API responses (password hash, etc.)
- [ ] Transactions used for multi-step writes

### Error Handling
- [ ] Global error handler catches all async errors
- [ ] Errors logged with context (not just message string)
- [ ] HTTP status codes correct and consistent
- [ ] Validation errors return field-level details (not just "invalid input")

---

## Supabase Review Checklist

### Row Level Security (RLS)
- [ ] RLS **enabled** on every table — not just some tables
- [ ] Policies cover all operations: `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- [ ] `auth.uid()` used in policies — not client-passed user ID
- [ ] No policy uses `TRUE` (allows all) on write operations unless intentional public table
- [ ] Service Role key is NEVER exposed to the client (only anon key)
- [ ] `SECURITY DEFINER` functions have explicit search_path set
- [ ] Realtime subscriptions restricted by RLS (not just UI-level)

### Auth
- [ ] Email confirmation enabled for production
- [ ] OAuth providers configured with correct redirect URLs for prod/staging
- [ ] `getUser()` used server-side — not `getSession()` (session can be spoofed)
- [ ] JWT expiry configured appropriately (default 1 hour is often fine)
- [ ] Auth state change handled on client (`onAuthStateChange`)

### Data Access Patterns
- [ ] No direct `.from()` calls with user-provided table names
- [ ] Edge Functions used for business logic requiring elevated permissions
- [ ] Storage bucket policies match expected access (private vs public)
- [ ] Signed URLs used for sensitive file access — not public bucket URLs
- [ ] Database functions (`rpc()`) have appropriate security context

### Performance
- [ ] Indexes defined on frequently queried + filtered columns
- [ ] `.select()` specifies only needed columns (not `*` on large tables)
- [ ] Realtime enabled only on tables that need live updates
- [ ] Connection pooling configured (PgBouncer mode for serverless)

---

## tRPC Review Checklist

- [ ] Input schemas validated with Zod on every procedure
- [ ] `protectedProcedure` used for auth-required routes (not just middleware trust)
- [ ] Context properly typed and validated
- [ ] Error handling uses `TRPCError` with appropriate codes
- [ ] No sensitive data leaked in error messages
- [ ] Queries and mutations separated correctly (no mutations in queries)
- [ ] Subscription procedures scoped appropriately

---

## Prisma ORM Review Checklist

- [ ] Migrations tracked via `prisma migrate dev` — no ad-hoc `db push` in production
- [ ] `prisma.$transaction()` used for all multi-step writes
- [ ] No `findMany` without `take` (pagination limit) on large tables
- [ ] `select` specified on `findMany` / `findFirst` — avoid returning full objects
- [ ] `@unique` constraints defined in schema for business-unique fields
- [ ] Cascade deletes defined in schema (`onDelete: Cascade`), not in application code
- [ ] No raw SQL (`$queryRaw`) without parameterized inputs — injection risk
- [ ] `include` used carefully — no unbounded nested includes (N+1 equivalent)
- [ ] Schema file committed and `prisma generate` run in CI before build
- [ ] `DATABASE_URL` from environment — never hardcoded
- [ ] Connection pooling configured for serverless (PgBouncer or Prisma Accelerate)
- [ ] `prisma.disconnect()` called in serverless environments (Lambda, Edge Functions)

---

## Drizzle ORM Review Checklist

- [ ] Schema migrations are tracked and versioned (`drizzle-kit`)
- [ ] No raw SQL strings — use Drizzle query builder
- [ ] Transactions used for multi-step writes
- [ ] Indexes defined in schema (not just in DB ad-hoc)
- [ ] `prepared` statements used for frequently-run queries
- [ ] Nullable columns handled explicitly in TypeScript types

---

## Clerk Auth Review Checklist

- [ ] `auth()` / `currentUser()` called server-side only
- [ ] `clerkMiddleware()` configured in `middleware.ts`
- [ ] Route matchers protect all non-public routes
- [ ] `userId` from Clerk used as foreign key in DB — not email
- [ ] Webhook verification uses `svix` signature check
- [ ] Organization / role-based access implemented via Clerk metadata if needed
- [ ] Public key verified — not trusting `userId` from client request body

---

## Convex Backend Review Checklist

- [ ] Mutations have proper argument validation (`v.string()`, `v.id()`, etc.)
- [ ] Auth identity checked inside mutations — not just client-side
- [ ] Queries return only needed fields (not full document)
- [ ] Sensitive data excluded from public queries
- [ ] Indexes defined for frequent query patterns (`.withIndex()`)
- [ ] No heavy computation in queries — use `action` for external API calls
- [ ] File/storage access controlled by user identity
- [ ] Internal functions (`internalMutation`) used for privileged operations

---

## REST API Contract Review

### Design
- [ ] RESTful resource naming — nouns not verbs (`/orders`, not `/getOrders`)
- [ ] Versioning in path (`/api/v1/`)
- [ ] HTTP methods used correctly (GET/POST/PUT/PATCH/DELETE)
- [ ] Consistent plural naming

### Request / Response
- [ ] Request body validated with schema
- [ ] Response schema consistent across all endpoints
- [ ] Timestamps in ISO 8601 format
- [ ] IDs not exposed as sequential integers (use UUID or CUID)
- [ ] Pagination: cursor-based preferred for large datasets

### Status Codes

| Scenario | Correct Code |
|---------|-------------|
| Success create | 201 Created |
| Success read | 200 OK |
| Validation error | 400 Bad Request |
| Auth missing | 401 Unauthorized |
| Wrong permissions | 403 Forbidden |
| Not found | 404 Not Found |
| Rate limited | 429 (+ `Retry-After` header) |
| Server error | 500 (generic message only) |

---

## Code Review Output Format

```markdown
## Code Review — [File/PR/Feature]
**Reviewer:** [OWNER_NAME]
**Date:** [YYYY-MM-DD]
**Stack:** [Detected stack]

### Summary
[1–2 sentences: overall code health and main concern]

### 🔴 Must Fix (blocks merge)
1. **[Issue title]** — `[file:line]`
   - Problem: [what's wrong]
   - Risk: [why it matters]
   - Fix: [suggested correction with code snippet if helpful]

### 🟠 Should Fix (address before release)
1. **[Issue title]** — `[file:line]`
   - [description + fix]

### 🟡 Suggestions (non-blocking improvements)
1. [Description]

### ✅ Looks Good
- [What was done well — always include this section]

### Verdict
🔴 REQUEST CHANGES / 🟡 APPROVE WITH COMMENTS / 🟢 APPROVED
```

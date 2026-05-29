# CLAUDE.md — Pitch2PM

> Context for AI coding assistants. Keep this under 200 lines. Update it when you learn something non-obvious.

---

## What This Project Does

Bug-bounty-style idea marketplace. Companies post problem bounties. Innovators pitch solutions. PMs review and pay out. Built on Next.js 14 App Router + PostgreSQL + Stripe Connect.

---

## Commands

```bash
# Install deps
npm install

# Start dev server
npm run dev                    # http://localhost:3000

# Database
npx prisma migrate dev         # run pending migrations
npx prisma migrate dev --name <name>  # new migration
npx prisma studio              # visual DB browser
npx prisma db seed             # seed with test data

# Tests
npm test                       # Vitest (unit + integration)
npm run test:watch             # watch mode
npm run test:e2e               # Playwright
npm run test:e2e:ui            # Playwright with UI

# Type check
npm run typecheck              # tsc --noEmit

# Lint + format
npm run lint                   # ESLint
npm run format                 # Prettier

# Build
npm run build
```

---

## Workflow

**Start every session by running tests first.**

```
1. npm test          ← must be green before you touch anything
2. Read TODO.md      ← pick the next unchecked task
3. Write failing test (red)
4. Implement until test passes (green)
5. npm run typecheck && npm run lint
6. Review diff manually
7. Commit with descriptive message
8. Update TODO.md (check the box)
9. If you learned something: update CLAUDE.md or AGENTS.md
```

---

## Directory Map

```
src/
  app/                    # Next.js App Router
    (auth)/               # Login, register pages (no layout chrome)
    (dashboard)/          # Authenticated app shell
      company/            # PM-side: bounties, review queue
      innovator/          # Innovator-side: pitches, profile
      admin/              # Platform admin (restricted)
    (public)/             # Unauthenticated pages: bounty board, profiles
    api/                  # Route Handlers
      auth/               # NextAuth handler
      bounties/           # Bounty CRUD
      pitches/            # Pitch CRUD + review
      upload/             # R2 presigned URL
      webhooks/           # Stripe webhook
  components/             # Shared UI components (use shadcn/ui conventions)
  lib/                    # Pure utilities: formatters, validators, constants
  hooks/                  # Client-side React hooks
  types/                  # TypeScript interfaces that aren't Prisma types
  server/
    api/                  # Thin route handler helpers (auth check, error wrap)
    db/                   # Prisma client singleton (src/server/db/client.ts)
    services/             # Business logic (no HTTP concerns here)
      bounty.service.ts
      pitch.service.ts
      review.service.ts
      payment.service.ts
      email.service.ts

prisma/
  schema.prisma           # Source of truth for DB schema
  migrations/             # Never edit these manually
  seed.ts                 # Dev seed data

tests/
  unit/                   # Fast, no DB: services with mocked Prisma
  integration/            # Real DB (test database): full service + DB tests
  e2e/                    # Playwright: full browser flows
```

---

## Key Conventions

### Prisma
- Always use `src/server/db/client.ts` singleton — never `new PrismaClient()` inline
- Wrap multi-step DB ops in `prisma.$transaction([...])`
- Never expose Prisma types directly to the client — map to plain TS types in service layer

### API Routes (Next.js)
- Always check `getServerSession()` at the top of every route handler
- Return `NextResponse.json({ error: "..." }, { status: 4xx })` for client errors
- All errors caught and logged — never expose stack traces to client

### Services
- Services take plain arguments (not `Request` objects) — they're testable without HTTP
- Services throw typed errors (`class BountyNotFoundError extends Error`)
- Callers (route handlers) translate service errors to HTTP status codes

### File Uploads
- Use presigned R2 URLs — client uploads directly to R2, never through Next.js server
- Validate file type and size before issuing presigned URL (not after)
- Store R2 key in DB, generate fresh signed URL on each read (15-min expiry)

### Payments
- Never log Stripe secret keys or webhook signing secrets
- Always verify Stripe webhook signature before processing
- Use idempotency keys on Stripe API calls

### Testing
- Unit tests mock Prisma with `vitest-mock-extended`
- Integration tests use a separate `TEST_DATABASE_URL` (never production)
- E2E tests seed a predictable set of fixtures before each test file

---

## Non-Obvious Gotchas

- **App Router + NextAuth**: Use `getServerSession(authOptions)` in server components/route handlers. The `useSession()` hook only works in client components.
- **Prisma + Next.js dev**: Use the singleton pattern to avoid "too many connections" during hot reload.
- **R2 CORS**: Must configure CORS on the R2 bucket to allow PUT from your domain. Easy to forget in new environments.
- **Stripe Connect**: Test mode and live mode have separate sets of connected accounts. Don't mix them.
- **Status machine**: Pitch status transitions are one-way — validate in service layer, not just in the UI.

---

## Environment Variables

See `.env.example` for the full list. Critical ones:

| Key | Purpose |
|---|---|
| `DATABASE_URL` | Postgres connection (Neon/Supabase) |
| `TEST_DATABASE_URL` | Separate DB for integration tests |
| `NEXTAUTH_SECRET` | NextAuth JWT signing |
| `STRIPE_SECRET_KEY` | Stripe API |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signature verification |
| `RESEND_API_KEY` | Transactional email |
| `R2_ACCOUNT_ID` | Cloudflare R2 |
| `R2_ACCESS_KEY_ID` | Cloudflare R2 |
| `R2_SECRET_ACCESS_KEY` | Cloudflare R2 |
| `R2_BUCKET_NAME` | Cloudflare R2 |

# AGENTS.md — Pitch2PM

> Setup and coding standards for AI agents (Codex, Copilot, etc.)

---

## Setup

```bash
npm install
cp .env.example .env.local   # fill in required values
npx prisma migrate dev
npm test                      # must pass before making changes
```

Required environment variables for testing:
- `DATABASE_URL` — Postgres (use a local/test DB)
- `TEST_DATABASE_URL` — Separate DB for integration tests
- `NEXTAUTH_SECRET` — any 32-char random string in dev
- `STRIPE_SECRET_KEY` — Stripe test key (`sk_test_...`)
- `STRIPE_WEBHOOK_SECRET` — from Stripe CLI (`stripe listen`)

---

## Testing

**Always write the failing test before writing implementation code.**

```bash
npm test                  # run all Vitest tests
npm run test:watch        # watch mode during development
npm run test:e2e          # Playwright (requires dev server running)
```

Test file conventions:
- Unit tests: `tests/unit/<service-name>.test.ts`
- Integration tests: `tests/integration/<feature>.test.ts`
- E2E tests: `tests/e2e/<flow-name>.spec.ts`

For unit tests, mock Prisma with `vitest-mock-extended`:
```ts
import { mockDeep } from 'vitest-mock-extended'
import type { PrismaClient } from '@prisma/client'

const prismaMock = mockDeep<PrismaClient>()
```

For integration tests, use `TEST_DATABASE_URL` and reset between test suites:
```ts
beforeEach(async () => {
  await prisma.$executeRaw`TRUNCATE TABLE pitches, bounties, users CASCADE`
})
```

---

## Code Style

### TypeScript
- Strict mode enabled (`"strict": true` in tsconfig)
- No `any` — use `unknown` and type-narrow
- Prefer `type` over `interface` for union types; use `interface` for object shapes
- Exported types go in `src/types/` or colocated with their module

### React / Next.js
- Default to **Server Components** — only use `'use client'` when you need hooks or event handlers
- Co-locate component styles with component files (Tailwind classes only, no CSS modules)
- Use `shadcn/ui` components as base — don't rebuild what already exists
- Form handling: React Hook Form + Zod for validation

### Services
```ts
// ✅ Good — service is testable, no HTTP coupling
export async function createBounty(
  input: CreateBountyInput,
  authorId: string
): Promise<Bounty> { ... }

// ❌ Bad — HTTP concerns leak into business logic
export async function createBounty(req: NextRequest): Promise<NextResponse> { ... }
```

### Error handling
```ts
// Typed errors — throw in services, catch in route handlers
export class PitchRateLimitError extends Error {
  constructor() { super('Rate limit exceeded: max 5 active pitches per 24h') }
}

// Route handler maps to status code
} catch (err) {
  if (err instanceof PitchRateLimitError) return NextResponse.json({ error: err.message }, { status: 429 })
  throw err  // let Next.js handle unexpected errors
}
```

### Database
- All DB access through `src/server/db/client.ts` singleton
- Use `prisma.$transaction` for multi-step writes
- Add DB indexes for any column used in `WHERE` or `ORDER BY` in production queries

---

## PR Instructions

Before opening a PR:
1. `npm test` — all tests pass
2. `npm run typecheck` — zero TypeScript errors
3. `npm run lint` — zero lint errors
4. Manual review of the diff — you own every line

PR description must include:
- **What**: one-sentence summary of the change
- **Why**: link to TODO.md task or issue
- **Evidence**: paste test output OR screenshot for UI changes
- **Risk**: any areas where you're uncertain

Do not:
- Refactor code outside the scope of your task
- Remove or skip tests to make CI pass
- Add dependencies without discussing in the PR description
- Merge without human review

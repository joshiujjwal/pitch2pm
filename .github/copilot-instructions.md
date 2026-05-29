# Copilot Instructions — Pitch2PM

## Project Overview

Pitch2PM is a **bug-bounty-style idea marketplace**: companies post problem-space bounties, innovators submit structured pitches, PMs review and pay out. Think HackerOne but for product ideas instead of security vulnerabilities.

**Stack**: Next.js 14 (App Router), TypeScript (strict), PostgreSQL, Prisma ORM, NextAuth.js, Stripe Connect, Tailwind CSS, shadcn/ui, Vitest, Playwright.

---

## Coding Conventions

### General
- TypeScript strict mode — no `any`, no type assertions without comments explaining why
- File names: `kebab-case.ts` for modules, `PascalCase.tsx` for React components
- Barrel exports (`index.ts`) only in `src/types/` and `src/lib/` — not in `src/components/`

### Next.js App Router
- Default to **Server Components** — only add `'use client'` when using hooks/events
- Route handlers live in `src/app/api/` — keep them thin (auth check + call service + return response)
- Layouts wrap route groups: `(auth)`, `(dashboard)`, `(public)` — match the directory names

### Database (Prisma)
- Use the singleton: `import { prisma } from '@/server/db/client'`
- Multi-step writes always use `prisma.$transaction`
- When adding a new model: update schema → run migration → update seed → add service tests

### State machines (Pitch / Bounty status)
- Status transitions are enforced in the **service layer**, not just the UI
- Invalid transitions throw a typed error (e.g., `InvalidStatusTransitionError`)
- Document every allowed transition in a comment above the service function

### UI Components
- Use `shadcn/ui` primitives first — don't reinvent buttons, inputs, dialogs
- Tailwind classes only — no inline styles, no CSS modules
- Responsive by default: mobile-first breakpoints

---

## Testing Conventions

- Write the failing test **before** the implementation — always
- Unit tests mock Prisma with `vitest-mock-extended`
- Integration tests use `TEST_DATABASE_URL` and truncate tables in `beforeEach`
- E2E tests (Playwright) cover happy paths and key error states
- Test file mirrors source file: `src/server/services/bounty.service.ts` → `tests/unit/bounty.service.test.ts`

---

## Boundaries — Do NOT Do These Without Being Asked

- Do **not** refactor files outside the scope of the current task
- Do **not** remove or modify existing passing tests
- Do **not** add new npm dependencies without listing them in your response first
- Do **not** change the Prisma schema without also writing the corresponding migration
- Do **not** store secrets in source code — use environment variables
- Do **not** expose Prisma model types directly to client components — map through service layer types
- Do **not** use `console.log` in production code — use a proper logger or remove after debugging

---

## Key Files to Know

| File | Purpose |
|---|---|
| `prisma/schema.prisma` | Source of truth for all data models |
| `src/server/db/client.ts` | Prisma singleton — import from here |
| `src/server/services/` | All business logic lives here |
| `src/app/api/` | Route handlers (thin HTTP layer) |
| `src/lib/errors.ts` | Typed error classes |
| `src/types/index.ts` | Shared TypeScript types |
| `docs/spec.md` | Feature spec — read before implementing anything new |
| `TODO.md` | Current task list with evidence gates |

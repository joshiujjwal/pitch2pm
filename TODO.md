# Pitch2PM — Task Breakdown

## How to Use This File

Work one task at a time. For each task:
1. **Write tests FIRST** (red phase — tests must fail before you write code)
2. **Implement** until tests pass (green phase)
3. **Review the diff** manually before committing
4. **Commit** with a descriptive message
5. **Update** CLAUDE.md or AGENTS.md if you discovered something non-obvious

Each phase is an evidence gate — don't advance until tests pass and a human has reviewed.

---

## Phase 0: Foundation ⬜

- [ ] Init Next.js 14 with TypeScript and Tailwind: `npx create-next-app@latest . --typescript --tailwind --app --src-dir`
- [ ] Install and configure Prisma: `npm install prisma @prisma/client`, `npx prisma init`
- [ ] Install Vitest + React Testing Library: `npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/user-event`
- [ ] Install Playwright: `npm install -D @playwright/test`, `npx playwright install`
- [ ] Add `npm test` and `npm run test:e2e` scripts to `package.json`
- [ ] Write and pass first smoke test: `tests/unit/smoke.test.ts` — assert `1 + 1 === 2`
- [ ] Set up GitHub Actions CI: `.github/workflows/ci.yml` — runs `npm test` on push/PR
- [ ] Create `.env.example` with all required keys (DATABASE_URL, NEXTAUTH_SECRET, STRIPE_SECRET_KEY, RESEND_API_KEY, R2_*)
- [ ] Review all AI config files (CLAUDE.md, AGENTS.md, copilot-instructions.md) for accuracy

**Evidence gate**: CI passes, smoke test green ✅

---

## Phase 1: Data Model & Auth ⬜

- [ ] Write failing tests for Prisma schema validation (User, Company, Bounty, Pitch models)
- [ ] Define Prisma schema in `prisma/schema.prisma`:
  - `User` (id, email, name, role: INNOVATOR | PM | ADMIN, linkedinUrl, avatarUrl, createdAt)
  - `Company` (id, name, domain, logoUrl, verified, createdAt)
  - `CompanyMember` (userId, companyId, role: OWNER | PM | VIEWER)
  - `Bounty` (id, companyId, title, problemStatement, reward, currency, deadline, status: OPEN | CLOSED | PAUSED, tags[], createdAt)
  - `Pitch` (id, bountyId?, authorId, title, summary, deckUrl, status: DRAFT | SUBMITTED | UNDER_REVIEW | ACCEPTED | REJECTED, score, feedback, createdAt)
  - `PitchReview` (id, pitchId, reviewerId, score, feedback, createdAt)
  - `Transaction` (id, pitchId, amount, currency, stripePaymentIntentId, status, createdAt)
- [ ] Run `npx prisma migrate dev --name init`
- [ ] Set up NextAuth.js with GitHub + LinkedIn providers in `src/app/api/auth/[...nextauth]/route.ts`
- [ ] Write integration test: user can sign in (mock OAuth) and session is returned
- [ ] Protect `/dashboard` route — redirect to `/login` if unauthenticated
- [ ] Write e2e test: unauthenticated user hitting `/dashboard` lands on `/login`

**Evidence gate**: Auth tests green, Prisma migration clean ✅

---

## Phase 2: Bounty Board (Company Side) ⬜

- [ ] Write failing tests for `BountyService.create()`, `BountyService.list()`, `BountyService.close()`
- [ ] Implement `src/server/services/bounty.service.ts` with full CRUD
- [ ] Build `/app/(dashboard)/company/bounties/new` — form to create a bounty (title, problem statement, reward, deadline, tags)
- [ ] Build `/app/(dashboard)/company/bounties` — list view of company's bounties with status badges
- [ ] Build `/app/(dashboard)/company/bounties/[id]` — detail view showing submitted pitches
- [ ] Write integration tests: POST creates bounty in DB, GET returns paginated list
- [ ] Write e2e test: PM logs in → creates bounty → sees it in list
- [ ] Implement bounty status transitions: OPEN → PAUSED → CLOSED

**Evidence gate**: Bounty CRUD tests green, e2e flow passes ✅

---

## Phase 3: Pitch Submission (Innovator Side) ⬜

- [ ] Write failing tests for `PitchService.create()`, `PitchService.submit()`, `PitchService.listByAuthor()`
- [ ] Implement `src/server/services/pitch.service.ts`
- [ ] Build `/app/(dashboard)/innovator/pitches/new` — multi-step pitch form:
  - Step 1: Select bounty or "open pitch"
  - Step 2: Title + 500-char elevator pitch
  - Step 3: Full summary (markdown editor)
  - Step 4: Upload deck (PDF/PPT → R2)
  - Step 5: Review + submit
- [ ] Build `/app/(dashboard)/innovator/pitches` — list with status badges
- [ ] Implement R2 upload in `src/server/api/upload.ts` (presigned URL pattern)
- [ ] Write unit tests for file upload validation (type, size limits)
- [ ] Write e2e test: innovator submits a pitch against a bounty → status shows SUBMITTED

**Evidence gate**: Pitch submission tests green, file upload working ✅

---

## Phase 4: PM Review Workflow ⬜

- [ ] Write failing tests for `PitchReviewService.submit()`, scoring logic (1–10 scale)
- [ ] Implement `src/server/services/review.service.ts`
- [ ] Build `/app/(dashboard)/company/pitches/[id]/review` — review panel:
  - Score slider (1–10)
  - Feedback textarea (min 50 chars)
  - Accept / Reject / Request Info buttons
- [ ] Implement pitch status state machine: SUBMITTED → UNDER_REVIEW → ACCEPTED | REJECTED
- [ ] Send email via Resend when pitch status changes (use `src/server/services/email.service.ts`)
- [ ] Write integration test: accepting a pitch triggers status update + email mock called
- [ ] Write e2e test: PM reviews pitch → accepts → innovator's dashboard shows ACCEPTED

**Evidence gate**: Review workflow tests green, email service mocked ✅

---

## Phase 5: Payments (Bounty Payouts) ⬜

- [ ] Write failing tests for `PaymentService.createPaymentIntent()`, `PaymentService.confirmPayout()`
- [ ] Implement Stripe Connect flow in `src/server/services/payment.service.ts`
- [ ] Build `/app/(dashboard)/innovator/onboarding/stripe` — Stripe Connect onboarding for innovators
- [ ] Trigger payout when PM accepts pitch: create Stripe Transfer to innovator's connected account
- [ ] Implement webhook handler at `/app/api/webhooks/stripe/route.ts` for `payment_intent.succeeded`
- [ ] Write unit tests for webhook signature verification
- [ ] Write integration test: mock Stripe confirms payment → Transaction record created

**Evidence gate**: Payment flow tests green (Stripe test mode) ✅

---

## Phase 6: Discovery & Public Feed ⬜

- [ ] Build `/app/(public)/bounties` — public bounty board with filters (tag, reward range, company, deadline)
- [ ] Implement full-text search with Postgres `tsvector` on bounty title + problem statement
- [ ] Build company profile pages `/app/(public)/company/[slug]`
- [ ] Build innovator profile pages `/app/(public)/u/[username]` (accepted pitches, win rate)
- [ ] Write tests for search ranking and filter combinations
- [ ] Add pagination and infinite scroll to bounty board

**Evidence gate**: Search returns correct results, profile pages render ✅

---

## Phase 7: Polish & Harden ⬜

- [ ] Add rate limiting on pitch submission (max 5 active pitches per user per day)
- [ ] Add input sanitization on all markdown fields (DOMPurify server-side)
- [ ] Implement NDA flow: accepted pitch triggers e-signature request (HelloSign/Docusign API)
- [ ] Add admin dashboard: `/app/(admin)/` — verify companies, moderate bounties, view transactions
- [ ] Accessibility audit: run axe-core on all pages
- [ ] Lighthouse CI: all pages must score ≥ 90 performance, ≥ 95 accessibility
- [ ] Load test: `k6` script simulating 500 concurrent users browsing bounty board

**Evidence gate**: All audits pass, load test p99 < 500ms ✅

---

## Phase 8: Ship ⬜

- [ ] Deploy to Vercel (production + preview environments)
- [ ] Set up Neon (PostgreSQL) production database with connection pooling
- [ ] Configure Cloudflare R2 production bucket with CORS
- [ ] Set up Stripe production keys + webhook endpoints
- [ ] Add Sentry for error tracking
- [ ] Set up PostHog for product analytics
- [ ] Write runbook in `docs/runbook.md`
- [ ] Smoke test production deployment end-to-end

**Evidence gate**: Production deploy passes smoke tests ✅

---

## Parking Lot 🅿️

- Mobile app (React Native / Expo)
- AI-assisted pitch scoring (LLM rates pitches before PM sees them)
- Reputation system: innovator badges, PM response rate scores
- Team pitches: multiple innovators co-author a pitch
- Private bounties: invite-only pitch calls for enterprise
- Pitch analytics: PM sees deck view heatmaps
- Notification center (in-app + push)

---

## Lessons Learned 📝

_Add entries here as you discover non-obvious things. These feed back into CLAUDE.md._

- 

# Pitch2PM — Feature Specification

**Version**: 0.1 (pre-implementation)
**Status**: Draft
**Author**: TBD
**Last updated**: 2025-01

---

## 1. Problem Statement

Product Managers at top companies struggle to source novel, externally-validated ideas. At the same time, entrepreneurs and domain experts have valuable product insights but no structured channel to reach decision-makers. Cold emails get ignored. Accelerators are slow. There is no purpose-built marketplace connecting innovators with the companies that should be building their ideas.

Pitch2PM creates a structured, incentivized pipeline: companies post problem-space bounties, innovators submit pitches, and PMs evaluate them through a transparent review workflow with real financial upside on both sides.

---

## 2. User Personas

| Persona | Role | Goal |
|---|---|---|
| **Innovator** | Entrepreneur, researcher, domain expert | Submit pitches, earn bounties, get visibility |
| **PM / Decision Maker** | Product Manager at a company | Find novel ideas, evaluate submissions, source talent |
| **Company Admin** | Startup founder, Head of Product | Set up company profile, post bounties, manage team PMs |
| **Platform Admin** | Pitch2PM staff | Verify companies, moderate content, handle disputes |

---

## 3. Functional Requirements

### 3.1 Authentication & Profiles

- [ ] Users can sign up/in with GitHub, Google, or LinkedIn OAuth
- [ ] Users choose a role at registration: **Innovator** or **PM** (can hold both)
- [ ] Innovators can fill out: bio, LinkedIn URL, expertise tags, public pitch history
- [ ] PMs must be associated with a verified company before posting bounties
- [ ] Companies are verified by Platform Admin (domain email check + manual review)

### 3.2 Bounty Management (Company Side)

- [ ] Company Admin can create a **Bounty** with:
  - Title (max 100 chars)
  - Problem statement (markdown, max 2000 chars)
  - Reward amount + currency (USD / EUR / USDC)
  - Deadline (required, must be ≥ 7 days from now)
  - Tags (up to 10, from a curated taxonomy)
  - Visibility: Public | Private (invite-only)
- [ ] Bounties have status: `OPEN → PAUSED → CLOSED`
- [ ] PMs can close a bounty at any time (stops new submissions, existing pitches still reviewed)
- [ ] Companies can see all pitches submitted to their bounties in a review queue

### 3.3 Pitch Submission (Innovator Side)

- [ ] Innovators can submit a **Pitch** against:
  - A specific bounty (bounty pitch)
  - The open pool (no specific bounty, discoverable by any PM)
- [ ] Pitch form collects:
  - Title (max 100 chars)
  - Elevator pitch (max 500 chars — forces clarity)
  - Full summary (markdown, max 5000 chars)
  - Deck attachment (PDF or PPTX, max 20MB, stored on R2)
  - Optionally: prototype URL, GitHub repo, demo video URL
- [ ] Pitches start as `DRAFT`, must be explicitly `SUBMITTED`
- [ ] Innovators can edit a pitch while it is `DRAFT`; submitted pitches are locked
- [ ] Rate limit: max 5 active (non-rejected) pitches per user per rolling 24h

### 3.4 PM Review Workflow

- [ ] PMs see a paginated review queue for their company's bounties
- [ ] PM can change pitch status:
  - `SUBMITTED → UNDER_REVIEW` (claim for review, prevents duplicate review)
  - `UNDER_REVIEW → ACCEPTED` (triggers payout + NDA flow)
  - `UNDER_REVIEW → REJECTED` (requires written feedback, min 50 chars)
  - `UNDER_REVIEW → INFO_REQUESTED` (opens a private thread with the innovator)
- [ ] PM scores pitch on 3 axes (1–10 each): **Novelty**, **Feasibility**, **Strategic Fit**
- [ ] Only one PM can claim a pitch for review at a time (optimistic lock)
- [ ] Accepted pitches are hidden from other PMs on the same company

### 3.5 Payments

- [ ] Innovators complete Stripe Connect onboarding to receive payouts
- [ ] Companies escrow bounty reward when posting (held by Stripe)
- [ ] On acceptance: platform takes 15% fee, 85% transferred to innovator
- [ ] On bounty close with no accepted pitches: full escrow refunded to company
- [ ] Transaction history visible to both parties

### 3.6 Notifications

- [ ] Email (via Resend) on: pitch status change, new pitch on your bounty, payout confirmed
- [ ] In-app notification bell (polling or WebSocket)
- [ ] Weekly digest email: new bounties matching innovator's tags

### 3.7 Discovery

- [ ] Public bounty board: filterable by tag, company, reward range, deadline
- [ ] Full-text search on bounty title + problem statement (Postgres `tsvector`)
- [ ] Public company profiles: logo, verified badge, open bounties, accepted pitch count
- [ ] Public innovator profiles: accepted pitches, tags, win rate (accepted / submitted)

---

## 4. Non-Functional Requirements

- [ ] **Performance**: Bounty board page loads in < 1s (LCP) on fast 3G
- [ ] **Availability**: 99.9% uptime (Vercel + Neon SLA)
- [ ] **Security**: All API routes authenticated; pitch decks not publicly guessable (signed URLs, 15-min expiry)
- [ ] **Privacy**: Innovator's identity is hidden from PMs until pitch is ACCEPTED (pseudonymous review mode — optional toggle)
- [ ] **Accessibility**: WCAG 2.1 AA on all user-facing pages
- [ ] **Scalability**: Designed to handle 10,000 bounties + 100,000 pitches without schema changes

---

## 5. Data Model (Simplified)

```
User ──< CompanyMember >── Company ──< Bounty ──< Pitch
                                                     │
                                               PitchReview
                                                     │
                                               Transaction
```

Key constraints:
- A `Pitch` can have `bountyId = null` (open-pool pitch)
- A `PitchReview` is 1:1 with a `Pitch` (one reviewer per pitch per company)
- A `Transaction` is created only when status → `ACCEPTED`

---

## 6. API Surface (Route Handlers)

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/api/bounties` | Public | List bounties (paginated, filterable) |
| POST | `/api/bounties` | PM | Create bounty |
| GET | `/api/bounties/[id]` | Public | Get bounty detail |
| PATCH | `/api/bounties/[id]` | PM (owner) | Update bounty status |
| POST | `/api/pitches` | Innovator | Create/submit pitch |
| GET | `/api/pitches/[id]` | Auth | Get pitch (author or PM of linked bounty) |
| PATCH | `/api/pitches/[id]/review` | PM | Submit review (score + decision) |
| POST | `/api/upload/presign` | Auth | Get presigned R2 URL for deck upload |
| GET | `/api/company/[slug]` | Public | Company profile |
| GET | `/api/users/[username]` | Public | Innovator profile |
| POST | `/api/webhooks/stripe` | Stripe sig | Handle payment events |

---

## 7. Test Plan

### Unit Tests (Vitest)
- `BountyService`: create, list (with filters), close, escrow calculation
- `PitchService`: create, submit, rate-limit enforcement, status machine transitions
- `ReviewService`: score validation, status lock (only one reviewer), feedback min-length
- `PaymentService`: fee calculation (85/15 split), payout amount, refund on close
- `EmailService`: correct template selected per event, recipient determined correctly

### Integration Tests (Vitest + Prisma test DB)
- Full pitch lifecycle: create → submit → review → accept → transaction created
- Bounty close with refund: post bounty → escrow → close → refund record
- Rate limit: 5th pitch succeeds, 6th returns 429

### E2E Tests (Playwright)
- Innovator signs up → submits pitch against bounty → sees SUBMITTED status
- PM logs in → reviews pitch → accepts → innovator dashboard shows ACCEPTED
- Unauthenticated user → browse public bounty board → click bounty → see detail

---

## 8. Open Questions

1. **Pseudonymous review**: Should innovator identity be hidden from PMs by default? What reveals it? (Acceptance only?)
2. **Dispute resolution**: If a company accepts a pitch and then ghosts on engagement, what recourse does the innovator have?
3. **NDA flow**: Integrate HelloSign or build lightweight in-app e-signature? What's MVP?
4. **Crypto payouts**: USDC on Base as alternative to Stripe? Later phase?
5. **Idea protection**: How do we prevent PMs from "noting" an idea internally without accepting? (Timestamp + hash of pitch at submission?)
6. **Pitch expiry**: Should unreviewed pitches expire after 90 days?

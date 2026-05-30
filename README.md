# Pitch2PM 🎯

> Connect entrepreneurs and innovators directly with product decision makers at top companies — bug-bounty-style idea marketplace.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-Next.js%20%7C%20TypeScript%20%7C%20PostgreSQL-blue)

---

## What It Is

Pitch2PM is a platform where innovators post structured product ideas ("pitches") and companies post "idea bounties" — open calls for solutions in specific problem spaces. When a PM or decision-maker accepts a pitch, the bounty is paid out and both parties enter a formal engagement process.

**Core loop:**
1. Company posts an **Idea Bounty** (problem space + reward + criteria)
2. Innovator submits a **Pitch** against a bounty (or open-pool)
3. PM reviews, scores, and accepts/rejects
4. Accepted pitches trigger payout + optional NDA/licensing flow

---

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS + shadcn/ui |
| Database | PostgreSQL (via Neon or Supabase) |
| ORM | Prisma |
| Auth | NextAuth.js (GitHub + Google + LinkedIn OAuth) |
| Payments | Stripe Connect (bounty payouts) |
| Email | Resend |
| Storage | Cloudflare R2 (pitch decks/attachments) |
| Testing | Vitest + Playwright |
| Hosting | Vercel |

---

## Getting Started

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/pitch2pm.git
cd pitch2pm

# Install dependencies
npm install

# Set up environment
cp .env.example .env.local
# Edit .env.local with your DB, auth, Stripe, etc.

# Database setup
npx prisma migrate dev

# Run dev server
npm run dev

# Run tests
npm test
```

---

## Project Structure

```
pitch2pm/
├── src/
│   ├── app/                  # Next.js App Router pages & layouts
│   ├── components/           # Shared React components
│   ├── lib/                  # Utilities, helpers, constants
│   ├── hooks/                # Custom React hooks
│   ├── types/                # TypeScript interfaces & types
│   └── server/
│       ├── api/              # tRPC or Route Handler logic
│       ├── db/               # Prisma client, schema helpers
│       └── services/         # Domain services (pitches, bounties, payments)
├── tests/
│   ├── unit/                 # Pure function + service tests
│   ├── integration/          # API + DB integration tests
│   └── e2e/                  # Playwright browser tests
├── docs/
│   ├── spec.md               # Feature specification
│   └── adr/                  # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   └── skills/
├── README.md
├── TODO.md
├── CLAUDE.md
└── AGENTS.md
```

---

## Contributing

- **Red/Green TDD**: Write failing tests first, then implement. No merging untested code.
- **Small PRs**: One feature or fix per PR. Keep diffs reviewable.
- **Evidence required**: PR descriptions must include test output, screenshots, or a demo clip for UI changes.
- **No AI slop**: Review every AI-generated diff before committing. You own the code.
- **ADRs for big decisions**: Architectural changes need a doc in `docs/adr/`.

---

## 🚀 Improvement Proposals

### First-Principles Analysis
- **The fundamental problem is bilateral market bootstrapping**: companies won't post bounties if there are no quality pitches, and innovators won't submit pitches if there are no active bounties — both sides must reach critical mass simultaneously, which is the hardest problem in marketplace design.
- **The "idea bounty" model assumes companies can specify their problem space well enough to attract relevant pitches** — in practice, product teams often don't know what they want until they see it; a more open "inspiration pitch" flow (innovator posts, PM discovers) may convert better initially.
- **Intellectual property is the core legal risk, not payments** — once a PM reads a pitch, the company has been "exposed" to the idea regardless of whether they accept; without a clear legal framework (timestamped submissions, NDA triggers), the platform is a liability magnet for both sides.
- **Stripe Connect for bounty payouts requires identity verification (KYC) for both payers and payees** — this is a significant onboarding friction that will kill conversion; the README does not address this, but it will be the first production blocker.

### Key Risks & Assumptions
- **Assumes companies will pay real money for unsolicited external ideas** — most large companies have legal policies against accepting unsolicited IP to avoid contamination claims; the target customer is more likely a startup or indie PM, not an enterprise product team.
- **Assumes the review and scoring workflow can be lightweight** — PMs are time-constrained; if reviewing a pitch takes more than 5 minutes, rejection rates will be near 100 % and innovators will stop submitting.
- **No mention of idea quality filtering or spam prevention** — an open submission pool will quickly fill with low-effort or AI-generated pitches; without a curation layer, the signal-to-noise ratio destroys company-side value.
- **The optional NDA/licensing flow is marked as a post-acceptance step**, but companies need legal clarity before they read a pitch, not after — the sequencing is legally backwards.

### Concrete Improvement Ideas
1. **Add pre-submission NDA click-wrap with timestamped submission receipts** — generate a PDF record of each pitch with a cryptographic timestamp at submission; this is the single highest-impact trust feature for both sides and the primary legal protection for innovators.
2. **Build a "PM Discovery Feed" as the primary company interface** — instead of requiring companies to post bounties first, let PMs browse an open pitch feed filtered by domain/tag; lowers company onboarding friction and breaks the cold-start problem.
3. **Add AI-powered pitch scoring before human review** — score pitches on clarity, novelty, and feasibility using an LLM; only surface top-quartile pitches to PMs; this protects PM time and increases company-side conversion.
4. **Implement an escrow model for bounties** — company deposits bounty into escrow when posting; innovator sees the amount is locked before submitting; this removes payment uncertainty and dramatically improves innovator trust.
5. **Start with a single vertical (e.g., developer tools or fintech)** — domain-specific pitches are easier to evaluate, attract higher-quality innovators, and let you build a reference customer base before going horizontal.
6. **Add a "pitch preview" mode** — let companies see a blinded abstract (no contact info, no detailed claims) before triggering the full NDA/reveal flow; reduces legal exposure and increases browse-to-engage conversion.

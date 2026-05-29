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

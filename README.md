# Spec Hunter — Web App

Mobile-first PWA for wholesale laptop intake and grading. Create hardware records, run 8 interactive tests in the browser, upload photos from phone, get auto-grades and pricing suggestions.

**For JM505 shop. Built with Next.js + Firebase.**

## Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router) |
| Styling | Tailwind CSS |
| Icons | Lucide React |
| Database | Firestore |
| Storage | Firebase Storage |
| Backend | Firebase Functions |
| Hosting | Firebase Hosting |

## Quick Start

```bash
npm install
cp .env.example .env   # Fill in Firebase config
npm run dev            # http://localhost:3000
```

## Project Map

See `SPEC.md` — SDD 6-part specification
See `CLAUDE.md` — Rules of engagement for Claude Code
See `TEST.md` — Test plan with 40+ test cases

## Phases

| Phase | What | Status |
|-------|------|--------|
| 1 | Web App MVP — PWA, all tests, photos, grading, pricing | Planning |
| 2 | USB Boot Collector — separate repo | Planning |
| 3 | Production — QR, marketplace pricing, multi-op | Future |

## Gamification

7 levels (Technician Trainee → Spec Hunter Legend), 9 badges, streaks, XP scoring. All UI uses Lucide icons — no emoji.

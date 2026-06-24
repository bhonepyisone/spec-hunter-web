# Phase 1.1 — Discuss: Firebase Project + Next.js Scaffold

## Goal
Create the Firebase project, initialize Next.js app with Tailwind + App Router, connect Firebase SDK, and verify the stack works end-to-end.

## Implementation Decisions

### Firebase Project
- **Project ID**: `spec-hunter` (or `spec-hunter-xxxxx` if taken)
- **Region**: `asia-southeast1` (Singapore — closest to Thailand/JM505 shop)
- **Services to enable**: Firestore, Firebase Storage, Firebase Functions, Firebase Hosting
- **Pricing**: Blaze plan (pay-as-you-go) — Firestore has generous free tier
- **Init via**: `firebase init` in project root

### Next.js
- **Version**: 14.x (stable, well-tested with Firebase)
- **Router**: App Router ✅ (already in CLAUDE.md)
- **Package manager**: npm (user's preference, no pnpm/yarn config)
- **Scaffold**: `npx create-next-app@latest . --typescript --tailwind --app`
- **Port**: 3000 (dev), Firebase Hosting for production

### Tailwind
- **Version**: Tailwind CSS 3 (stable, well-documented)
- **Config**: `tailwind.config.ts` with custom colors from design system
- **Dark mode**: class-based (toggle with `dark` class on `<html>`)

### Firebase SDK
- **Packages**: `firebase` (client), `firebase-admin` (Functions), `firebase-tools` (CLI)
- **Client init**: `lib/firebase.ts` — `initializeApp` with env vars
- **Env vars**: `.env.local` with Firebase config (apiKey, authDomain, projectId, storageBucket, messagingSenderId, appId)
- **No Auth SDK** needed — single operator, no login

### Project Structure (After Scaffold)
```
spec-hunter-web/
├── app/                    ← Next.js App Router pages
├── components/             ← React components
├── lib/                    ← Firebase init, helpers
├── types/                  ← TypeScript types
├── functions/              ← Firebase Functions (Node.js)
├── public/                 ← Static assets, PWA icons
├── .env.local              ← Firebase config
├── firebase.json           ← Firebase project config
├── .firebaserc             ← Firebase project alias
├── firestore.rules         ← Security rules
├── storage.rules           ← Storage rules
└── ... (existing planning files)
```

### Firebase Emulator
- **Use emulator?**: No — direct connection to production Firestore for MVP
- **Exception**: Firebase Functions tested via `firebase serve` or direct deploy

### Design System Integration
- **Colors**: Add to `tailwind.config.ts` — bg #020617, surface #1A1E2F, accent #22C55E
- **Fonts**: Import Fira Code + Fira Sans via Google Fonts in `layout.tsx`
- **Icons**: `lucide-react` package — NO emoji

## Decisions Made
| Decision | Choice | Rationale |
|----------|--------|-----------|
| Firebase region | `asia-southeast1` | Closest to Thailand, lowest latency |
| Next.js version | 14.x | Stable, proven Firebase integration |
| Package manager | npm | User's existing setup |
| Tailwind dark mode | class-based | More control than system-based |
| Firebase emulator | Skip for MVP | Direct to production is simpler |
| Firebase Functions | Node.js 20 | Latest LTS |

## Open Questions
- None — all decisions covered from existing planning files.

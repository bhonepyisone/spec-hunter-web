# Phase 1.1 — Plan: Firebase Project + Next.js Scaffold

## Task Summary
Set up Firebase project, scaffold Next.js app with Tailwind + App Router, wire Firebase SDK, and create first page to verify everything works.

## Files to Touch

| File | Action | Notes |
|------|--------|-------|
| `firebase.json` | Create | Firebase project config |
| `.firebaserc` | Create | Project alias |
| `firestore.rules` | Create | Copy from `LLM_Wiki/references/security-rules-draft.md` |
| `storage.rules` | Create | Copy from `LLM_Wiki/references/security-rules-draft.md` |
| `.env.local` | Create | Firebase config from console (DO NOT commit) |
| `lib/firebase.ts` | Create | Firebase init (client SDK) |
| `app/layout.tsx` | Edit | Add dark theme, fonts, metadata |
| `app/page.tsx` | Edit | Simple test page |
| `tailwind.config.ts` | Edit | Add design system colors + fonts |
| `next.config.ts` | Edit | PWA config (minimal) |

## Acceptance Criteria
1. `npm run dev` starts without errors
2. `http://localhost:3000` shows a dark-themed page with "Spec Hunter" title
3. Firestore connection works: open browser console → `db` object exists
4. Firebase config loaded from `.env.local`
5. Fira Code + Fira Sans fonts render on page
6. Lucide icon renders on page (test: import and render `Wrench` icon)

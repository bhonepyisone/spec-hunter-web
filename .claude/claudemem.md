# Claude Mem — Spec Hunter Web

## Durable Facts

- Single operator internal tool — NO auth/login
- Dark theme: bg #121317, surface #1a1b1f, accent #2e96ff
- NO emoji in UI — ALL icons from Lucide React
- Firebase only: Firestore + Storage + Functions + Hosting
- Battery thresholds: A+≥85, A≥75, B≥60, C≥40, D<40
- Pricing: buy_price × grade_multiplier (A+=1.45, A=1.35, B=1.20, C=1.05, D=0.85)
- 8 interactive tests in browser: keyboard, LCD, speaker, camera, mic, touchpad, USB, WiFi
- Gamification: 7 levels, 9 badges, XP scoring, streak
- Photo upload: phone camera via `<input capture="environment">`
- Read SPEC.md before any change — DoD items = commit checklist

## User Preferences

- Mobile-first, test at 375px minimum
- One component per file, default export
- TypeScript — no `any` without reason
- DO NOT auto-commit — user reviews and commits manually

## Project State

- Phase: Planning — all GSD artifacts created
- Next: Firebase project setup + first Claude Code prompt

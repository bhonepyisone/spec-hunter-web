# Agent: Spec Hunter Web Developer

## Role
Full-stack developer specializing in Next.js PWA + Firebase applications for laptop inspection and grading.

## Context
- **Project**: Spec Hunter — Laptop Intake & Grading System (Web App)
- **Stack**: Next.js 14+ (App Router), Tailwind CSS, Firestore, Firebase Storage, Firebase Functions, Lucide React
- **Rules**: CLAUDE.md is authoritative — read it first. Mobile-first responsive, dark theme (#121317 bg, #2e96ff accent), NO emoji in UI (Lucide icons only), single operator (no auth), formula-based grading and pricing.

## Constraints
- No emoji characters in any UI component — use `lucide-react` icons
- No authentication/login system — internal single-operator tool
- No PostgreSQL or SQL — Firestore only
- No QR code features in MVP
- No ML pricing — formula-based only (buy_price × grade_multiplier)
- All 8 interactive tests must use browser APIs (getUserMedia, AudioContext, Fullscreen API, keydown events)
- Photo upload via phone camera `<input capture="environment">`
- Gamification scoring: +10/laptop, +25/A+, +15/A, +5 standard, -5 skipped test
- 7 levels: Trainee(0) → Junior(100) → Inspector(250) → Senior(500) → Lead(1000) → Master(2000) → Legend(5000)
- Pre-commit hook must check for: staged .env files (block), TypeScript compilation (block on fail)
- DO NOT auto-commit — user reviews and commits manually
- **Before every commit, run the Review Agent first** — never commit without review

---

# Agent: Spec Hunter Code Reviewer

## Role
Senior engineer reviewing all code changes before commit. Catches bugs, mobile responsiveness issues, edge cases, gamification balance problems, and non-compliant patterns.

## When to Run
Before every `git commit`. After staging files with `git add`.

## How to Run
```bash
# Review all staged changes
claude -p 'Review staged changes for bugs, mobile responsiveness on 375px, edge cases (empty states, loading, errors), gamification balance, and compliance with CLAUDE.md rules (Lucide icons, dark theme, no auth, Firestore only). List all issues with file paths and line references.'
```

## Review Checklist
1. **Mobile responsiveness**: Does it work at 375px? Touch targets ≥44px?
2. **Dark theme**: bg #121317, surface #1a1b1f, accent #2e96ff?
3. **Icons**: Lucide only? Any emoji accidentally used?
4. **Edge cases**: Empty state? Loading state? Error state?
5. **Gamification**: Points awarded correctly? Level triggers right? Streak logic?
6. **Firestore**: Direct writes safe? No auth bypass?
7. **Tests**: All 8 interactive tests handle permission denial gracefully?
8. **Photo upload**: Handles cancel? Handle 11th photo?

## Constraints
- Output issues as structured list with file:line references
- Priority: crash bugs > data loss > visual bugs > edge cases
- If no issues found: output "✅ No issues found"
- DO NOT auto-fix — output issues for the Fix Agent

---

# Agent: Spec Hunter Fix Agent

## Role
Precise fixer that takes the Review Agent's issue list and applies targeted fixes.

## When to Run
After the Review Agent outputs issues. Read the issue list, fix each one.

## How to Run
```bash
claude -p 'Fix the following issues (paste Review Agent output here):
1. [issue 1] in components/tests/KeyboardTest.tsx:42
2. [issue 2] in lib/gamification.ts:88
...'
```

## Constraints
- One fix per issue. Do not refactor unrelated code.
- After fixing, verify the fix didn't break existing functionality.
- DO NOT auto-commit — stage fixes for user review.
- If a fix would require breaking the spec, flag it instead of fixing.

## Example Workflow
```
1. Developer: builds feature, runs `git add .`
2. Code Reviewer: `claude -p 'Review staged changes...'` → outputs 3 issues
3. Fix Agent: fixes all 3 issues
4. Developer: reviews fixes, `git diff`, verifies
5. Commit: `git add . && git commit -m "Add: feature + fix review issues"`
```

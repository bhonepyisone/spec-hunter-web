# Spec Hunter — Web App

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router) |
| Styling | Tailwind CSS |
| Icons | Lucide React (NO emoji anywhere) |
| Database | Firestore (NoSQL) |
| File Storage | Firebase Storage |
| Backend Logic | Firebase Functions (Node.js) |
| Hosting | Firebase Hosting |
| PWA | next-pwa or built-in Next.js PWA config |
| Runtime | Node.js 20+ |

## Project Structure

```
spec-hunter-web/
├── SPEC.md                     ← SDD 6-part spec (read this first)
├── CLAUDE.md                   ← This file
├── TEST.md                     ← Test plan
├── .mcp.json                   ← MCP tools
├── .gitignore
├── .env.example
├── .githooks/
│   └── pre-commit              ← Quality gate
├── .claude/
│   ├── skills/
│   │   └── spec-hunter-web/
│   │       └── SKILL.md
│   └── agents/
│       └── spec-hunter-web.md
├── LLM_Wiki/
│   ├── README.md
│   ├── architecture.md
│   └── api-docs.md
├── app/                        ← Next.js App Router
│   ├── page.tsx                ← Dashboard
│   ├── layout.tsx              ← Root layout (dark theme, PWA)
│   ├── laptops/
│   │   ├── page.tsx            ← Table view
│   │   ├── new/page.tsx        ← Entry form
│   │   └── [id]/
│   │       ├── page.tsx        ← Detail page
│   │       ├── test/page.tsx   ← Test suite
│   │       └── grade/page.tsx  ← Grade + pricing
│   └── profile/
│       └── page.tsx            ← Operator stats
├── components/
│   ├── dashboard/
│   │   ├── StatsCards.tsx
│   │   ├── StreakWidget.tsx
│   │   └── AchievementFeed.tsx
│   ├── tests/
│   │   ├── KeyboardTest.tsx
│   │   ├── LCDTest.tsx
│   │   ├── SpeakerTest.tsx
│   │   ├── CameraTest.tsx
│   │   ├── MicrophoneTest.tsx
│   │   ├── TouchpadTest.tsx
│   │   ├── USBTest.tsx
│   │   └── WiFiTest.tsx
│   ├── laptops/
│   │   ├── LaptopTable.tsx
│   │   ├── LaptopForm.tsx
│   │   ├── SpecsCard.tsx
│   │   └── PhotoGallery.tsx
│   ├── grade/
│   │   ├── GradeBadge.tsx
│   │   ├── PricingCard.tsx
│   │   └── BodyConditionForm.tsx
│   └── gamification/
│       ├── LevelProgressBar.tsx
│       ├── BadgeIcon.tsx
│       └── PointsAnimation.tsx
├── lib/
│   ├── firebase.ts             ← Firebase init
│   ├── firestore.ts            ← CRUD helpers
│   ├── grading.ts              ← Grade calculator
│   ├── pricing.ts              ← Price formula
│   ├── gamification.ts         ← Points, levels, badges
│   └── storage.ts              ← Photo upload to Firebase Storage
├── types/
│   └── laptop.ts               ← TypeScript types matching Firestore schema
├── functions/                  ← Firebase Functions
│   ├── src/
│   │   └── index.ts            ← uploadLaptop endpoint
│   ├── package.json
│   └── tsconfig.json
├── public/
│   ├── manifest.json           ← PWA manifest
│   ├── sw.js                   ← Service worker
│   └── icons/                  ← PWA icons (192x192, 512x512)
├── firebase.json
├── .firebaserc
├── firestore.rules
├── storage.rules
├── next.config.ts
├── tsconfig.json
├── tailwind.config.ts
└── package.json
```

## Conventions

### Design System (from UI-UX Pro Max)
- **Style**: Data-Dense Dashboard
- **Colors**: bg #020617, surface #1A1E2F, accent #22C55E, primary #0F172A, foreground #F8FAFC, border #334155, destructive #EF4444
- **Typography**: Fira Code (headings), Fira Sans (body)
- **Google Fonts**: `family=Fira+Code:wght@400;500;600;700&family=Fira+Sans:wght@300;400;500;600;700`
- **Effects**: `rounded-lg` (8px), `rounded-xl` (12px), `rounded-2xl` (16px), shadow-md, hover tooltips, row highlighting
- **Icons**: Lucide SVG icons — NO emoji anywhere in UI
- **Full spec**: `design-system/spec-hunter/MASTER.md`

### Code Style
- One component per file, default export
- TypeScript throughout — no `any` without explicit reason
- Tailwind utility classes only — no custom CSS files
- Lucide icons (`lucide-react`) — NO emoji characters in UI

### Theme
- Dark theme: bg `#121317`, surface `#1a1b1f`, accent `#2e96ff`
- All pages must render in dark theme by default
- Colors stored in Tailwind config as custom tokens

### Responsive Design
- Mobile-first with Tailwind breakpoints: sm:640, md:768, lg:1024, xl:1280
- Touch targets minimum 44x44px on mobile
- Body text minimum 16px on mobile
- Test on 375px width minimum
- All pages must work on mobile (PWA)

### Firebase
- Read/write from client-side Firestore SDK (no REST calls)
- Photos stored in Firebase Storage under `photos/{laptopId}/{fileName}`
- USB upload handled by Firebase Function with API key auth
- Security rules: write requires api-key header check (for Function), read is public (single operator internal tool)

### Gamification
- Points: +10/laptop, +25/A+ grade, +15/A grade, +5 standard pass, -5 skipped test
- Levels: 7 levels (Trainee → Legend), XP ranges: 0/100/250/500/1000/2000/5000
- Badges: First Grading, Perfect Scan, A+ Master, Speed Demon, Photo Pro, Daily Driver, Weekly Warrior, Century, Perfect Week
- Streak: days with at least 1 processed laptop, resets after 24h gap
- All gamification UI uses Lucide icons, NOT text badges or emoji

### Commits
- One commit per DoD item from SPEC.md
- User reviews and commits manually (DO NOT auto-commit)
- Commit messages: imperative tense, prefixed with category
  - `"Add: keyboard test component"`
  - `"Wire: photo upload to Firebase Storage"`
- Push after each commit

### Code Review Workflow (MANDATORY)
- **Must run Review Agent before every commit** — never commit without review
- Workflow:
  1. `git add <files>`
  2. Run Review Agent: load `.claude/agents/spec-hunter-web.md` → follow Review Agent instructions
  3. If issues found → run Fix Agent from the same file
  4. `git diff` → verify fixes
  5. `git commit`
- Review checks: mobile 375px, dark theme, Lucide icons, edge cases (empty/loading/error), gamification balance, Firestore safety, permission denial handling
- If Review Agent finds zero issues: output "✅ No issues found" → safe to commit

## DO NOT
- Commit `.env` files or real API keys
- Auto-commit — let user review and commit manually
- Use emoji in any UI element (Lucide icons only)
- Add auth/login — this is a single-operator internal tool
- Add PostgreSQL/SQL — use Firestore only
- Add QR code features in MVP
- Use any library not listed in SPEC.md

## Testing Requirements
- After each feature: run manual test in browser
- All 8 interactive tests must work on real hardware (camera, mic, speakers)
- Before major push: full E2E walkthrough (create laptop → all tests → photos → grade → pricing)
- Responsive: check at 375px (phone), 768px (tablet), 1280px (desktop)

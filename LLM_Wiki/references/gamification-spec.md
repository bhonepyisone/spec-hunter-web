# Gamification System — Complete Spec

> Consolidated reference for all gamification mechanics in Spec Hunter web app.
> Source of truth: this doc. Implement in `lib/gamification.ts` and gamification components.

---

## 1. Points System

### Base Points

| Action | Points | Trigger |
|--------|--------|---------|
| Laptop processed (all tests complete + grade assigned) | +10 | On grade save |
| A+ grade bonus | +25 | On A+ grade assignment |
| A grade bonus | +15 | On A grade assignment |
| Standard pass (B or C) | +5 | On B/C grade assignment |
| Repaired (D grade) | +0 | On D grade assignment |
| Test skipped (any test left pending) | -5 | On grade save if pending tests exist |

### Point Calculation

```typescript
function calculatePoints(laptop: LaptopData): number {
  let points = 10;  // Base: laptop processed

  // Grade bonus
  if (laptop.grade === "A+") points += 25;
  else if (laptop.grade === "A") points += 15;
  else if (laptop.grade === "B" || laptop.grade === "C") points += 5;
  // D = no bonus

  // Penalty for skipped tests
  const pendingTests = Object.values(laptop.tests)
    .filter(t => t.status === "pending").length;
  points -= pendingTests * 5;

  return Math.max(0, points);  // Floor at 0
}
```

### Where Points Are Stored

```typescript
// On the laptop document:
{
  ...laptopData,
  points_earned: 10,       // Points from this laptop
  badges_earned: ["first-grading"],  // Badges earned on this laptop
}

// In a separate stats document (for aggregate):
// collection: "stats", document: "operator"
{
  total_xp: 450,
  current_level: 3,
  current_title: "Inspector",
  total_laptops_processed: 22,
  streak: 5,
  last_activity_date: "2026-06-24",
  badges: ["first-grading", "perfect-scan", "daily-driver"],
  grade_distribution: { "A+": 2, "A": 8, "B": 7, "C": 4, "D": 1 }
}
```

---

## 2. Level System

### Level Table

| Level | Title | Icon (Lucide) | Icon Name | XP Required | Total XP to Reach |
|-------|-------|---------------|-----------|-------------|-------------------|
| 1 | Technician Trainee | `wrench` | Wrench | 0 | 0 |
| 2 | Junior Inspector | `binoculars` | Binoculars | 100 | 100 |
| 3 | Inspector | `clipboard-check` | ClipboardCheck | 250 | 350 |
| 4 | Senior Inspector | `badge-check` | BadgeCheck | 500 | 850 |
| 5 | Lead Grader | `shield` | Shield | 1000 | 1850 |
| 6 | Master Grader | `crown` | Crown | 2000 | 3850 |
| 7 | Spec Hunter Legend | `swords` | Swords | 5000 | 8850 |

### Level Calculation

```typescript
const LEVELS = [
  { level: 1, title: "Technician Trainee", xpRequired: 0, icon: "Wrench" },
  { level: 2, title: "Junior Inspector",   xpRequired: 100, icon: "Binoculars" },
  { level: 3, title: "Inspector",           xpRequired: 250, icon: "ClipboardCheck" },
  { level: 4, title: "Senior Inspector",    xpRequired: 500, icon: "BadgeCheck" },
  { level: 5, title: "Lead Grader",         xpRequired: 1000, icon: "Shield" },
  { level: 6, title: "Master Grader",       xpRequired: 2000, icon: "Crown" },
  { level: 7, title: "Spec Hunter Legend",  xpRequired: 5000, icon: "Swords" },
];

function getLevel(totalXp: number): { level: number; title: string; icon: string; progress: number } {
  // Find highest level reached
  let currentLevel = LEVELS[0];
  for (const lvl of LEVELS) {
    if (totalXp >= lvl.xpRequired) currentLevel = lvl;
  }

  // Progress to next level
  const nextLevel = LEVELS.find(l => l.level === currentLevel.level + 1);
  let progress = 100;  // Max level
  if (nextLevel) {
    const earnedInLevel = totalXp - currentLevel.xpRequired;
    const neededForNext = nextLevel.xpRequired - currentLevel.xpRequired;
    progress = Math.round((earnedInLevel / neededForNext) * 100);
  }

  return { ...currentLevel, progress };
}
```

### UI Requirements

- **Level badge** shown in dashboard header + profile page
- **Progress bar** (0-100%) below the badge showing progress to next level
- **Current level title** displayed prominently (e.g., "Inspector" as a badge label)
- **Next level preview** on hover: "Next: Senior Inspector (145/500 XP)"
- Icon rendered via `import { Wrench } from 'lucide-react'` — NOT emoji

---

## 3. Badge System

### Badge Table

| ID | Badge Name | Icon (Lucide) | Requirement | Earned On |
|----|-----------|---------------|-------------|-----------|
| `first-grading` | First Grading | `Award` | Grade your first laptop | First grade |
| `perfect-scan` | Perfect Scan | `ScanCheck` | All 8 tests pass on the same laptop | Any laptop |
| `a-plus-master` | A+ Master | `Gem` | Grade your first A+ laptop | First A+ |
| `speed-demon` | Speed Demon | `Zap` | Complete all tests + grade in under 5 minutes | Any laptop (track start→finish time) |
| `photo-pro` | Photo Pro | `Camera` | Upload 10 photos on a single laptop | Any laptop |
| `daily-driver` | Daily Driver | `Flame` | Maintain a 3-day streak | 3 consecutive days |
| `weekly-warrior` | Weekly Warrior | `CalendarCheck` | Maintain a 7-day streak | 7 consecutive days |
| `century` | Century | `Trophy` | Process 100 laptops total | Cumulative |
| `perfect-week` | Perfect Week | `Star` | All laptops this week passed (no rejects) | End of week |

### Badge Check Logic

```typescript
interface BadgeCheck {
  id: string;
  check: (laptop: LaptopData, stats: OperatorStats) => boolean;
}

const BADGE_CHECKS: BadgeCheck[] = [
  {
    id: "first-grading",
    check: (laptop, stats) => stats.total_laptops_processed >= 1,
  },
  {
    id: "perfect-scan",
    check: (laptop) => Object.values(laptop.tests).every(t => t.status === "pass"),
  },
  {
    id: "a-plus-master",
    check: (laptop) => laptop.grade === "A+",
  },
  {
    id: "speed-demon",
    check: (laptop) => {
      const startTime = laptop.created_at;  // Timestamp when laptop was created
      const gradeTime = laptop.updated_at;   // Timestamp when grade was saved
      return (gradeTime - startTime) < 5 * 60 * 1000;  // Under 5 minutes
    },
  },
  {
    id: "photo-pro",
    check: (laptop) => (laptop.photo_count ?? 0) >= 10,
  },
  {
    id: "daily-driver",
    check: (laptop, stats) => stats.streak >= 3,
  },
  {
    id: "weekly-warrior",
    check: (laptop, stats) => stats.streak >= 7,
  },
  {
    id: "century",
    check: (laptop, stats) => stats.total_laptops_processed >= 100,
  },
  {
    id: "perfect-week",
    check: (laptop, stats) => {
      // Check: in the current week (Mon-Sun), all laptops are "passed"
      // This requires querying laptops created this week and checking none are "rejected"
      return true;  // Placeholder — actual implementation queries Firestore
    },
  },
];
```

### Badge Display

- **Dashboard**: Recent badges shown in a horizontal scrollable row
- **Profile page**: All badges in a grid, earned ones colored, unearned ones grayed out with lock icon
- **Earned notification**: Brief animation when a new badge is earned (e.g., badge icon scales up + text "Badge Earned: A+ Master")

---

## 4. Streak System

### Mechanics

| Rule | Detail |
|------|--------|
| Definition | Consecutive days with at least 1 laptop graded (passed or rejected) |
| Storage | `stats.last_activity_date` (ISO date string: `YYYY-MM-DD`) |
| Increment | Each day the operator processes ≥1 laptop |
| Reset | Streak resets to 0 if 24h+ gap between activity dates |
| Display | Flame icon + number on dashboard (e.g., "5-day streak") |
| Milestones | Badges at 3 days (Daily Driver) and 7 days (Weekly Warrior) |

### Streak Calculation

```typescript
function updateStreak(stats: OperatorStats): number {
  const today = new Date().toISOString().split("T")[0];  // "2026-06-24"
  const lastActive = stats.last_activity_date;

  if (lastActive === today) {
    // Already processed today — streak stays the same
    return stats.streak;
  }

  const yesterday = new Date(Date.now() - 86400000).toISOString().split("T")[0];
  
  if (lastActive === yesterday) {
    // Processed yesterday — increment streak
    return stats.streak + 1;
  }

  if (lastActive && lastActive !== today) {
    // Gap detected — reset streak to 1 (today counts)
    return 1;
  }

  // First ever activity
  return 1;
}
```

---

## 5. Profile Page Layout

```
┌──────────────────────────────┐
│  [Level Badge]               │
│  🛡️ Lead Grader              │  ← Level badge + title
│  ███████████░░░░░ 72%        │  ← Progress bar to next level
│  Next: Master Grader         │
│                              │
│  Total XP: 1,850             │
│  Laptops: 142                │
│  Streak: 5 🔥                │  ← Streak counter with flame icon
│                              │
│  ── Grade Distribution ──    │
│  A+ ████░ 12  (8%)           │
│  A  ████████████░░ 45 (32%)  │  ← Bar chart per grade
│  B  ████████████████ 55(39%) │
│  C  ██████░ 22  (15%)        │
│  D  ███░░ 8   (6%)           │
│                              │
│  ── Badges (9) ──            │
│  [🏆] [✅] [💎] [⚡]        │  ← Earned badges (colored)
│  [📷] [🔥] [📅] [🔒] [🔒]  │  ← Locked badges (gray)
│                              │
│  Recent Achievements         │
│  • A+ Master — June 24       │
│  • Century — June 20         │
│  • Speed Demon — June 18     │
└──────────────────────────────┘
```

---

## 6. Dashboard Gamification Widgets

| Widget | Location | Content |
|--------|----------|---------|
| Level Badge | Top of dashboard | Current level icon + title + progress bar |
| Streak Counter | Next to stats | Flame icon + days count |
| Today's Score | Right column | "Today: +35 XP" |
| Recent Badges | Below stats | Horizontal scroll of last 3-5 earned badges |
| Next Milestone | Bottom | "5 more laptops until Century badge" |

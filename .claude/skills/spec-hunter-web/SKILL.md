---
name: spec-hunter-web
description: "Complete development workflows for the Spec Hunter web app — build interactive tests, wire Firebase, deploy, code review."
tags: [nextjs, firebase, pwa, grading, gamification, code-review]
---

# Spec Hunter Web — Development Skills

---

## Skill 1: Build an Interactive Test Component

When building any of the 8 interactive test components (keyboard, LCD, speaker, camera, mic, touchpad, USB, WiFi).

### Test Building Pattern

```typescript
// Each test component follows this interface:
interface TestProps {
  laptopId: string;
  onComplete: (result: TestResult) => void;
}

interface TestResult {
  status: "pass" | "fail" | "pending";
  // Test-specific data (e.g. failed_keys, thumbnail_url, latency_ms)
  [key: string]: any;
}
```

### Test API Reference

| Test | Browser API | Key Implementation Details |
|------|-------------|--------------------------|
| **Keyboard** | `keydown` events + SVG layout | Track pressed keys in a Set<string>. Compare against complete key list. Auto-PASS when all pressed. |
| **LCD** | `Fullscreen API` + `<div>` | Cycle colors with a state counter. Color order: white → black → red → green → blue → gray. Record defect type on FAIL. |
| **Speaker** | `AudioContext.createOscillator()` | Create OscillatorNode at 440Hz. Split to left/right channel using `channelInterpretation`. |
| **Camera** | `getUserMedia({ video: true })` | Show preview `<video>` element. Capture via `<canvas>.drawImage()`. Save blob to state, then upload to Firebase Storage. |
| **Microphone** | `MediaRecorder` | Stream → MediaRecorder → chunks → Blob → createObjectURL → playback via `<audio>`. |
| **Touchpad** | `pointermove` + `click` | Track `pointermove` events in an array. On click, check if pointer moved >10px total. |
| **USB Port** | Manual button click | Simple UI: "Plug USB drive and click Detected" → PASS button / FAIL button |
| **WiFi** | `fetch()` with AbortController | `fetch(url, { signal: AbortSignal.timeout(5000) })`. Measure elapsed time for latency. |

### Steps

1. Create component file in `components/tests/<TestName>.tsx`
2. Implement the TestProps interface
3. Use the browser API to test (reference table above)
4. On completion, call `onComplete({ status, ...data })`
5. Update Firestore: `updateDoc(docRef, { ['tests.testName']: result })`
6. Handle permission denial gracefully (show error message, allow manual override)
7. Verify on real hardware (camera/mic need real laptop, not dev machine)

---

## Skill 2: Wire Firebase Firestore + Storage

When setting up Firebase or reading/writing data.

### Steps

1. Init Firebase in `lib/firebase.ts`:
```typescript
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';
import { getStorage } from 'firebase/storage';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  // ... from .env.local
};

export const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
export const storage = getStorage(app);
```

2. CRUD helpers in `lib/firestore.ts`:
```typescript
// createLaptop(data) → addDoc
// getLaptop(id) → getDoc
// updateLaptop(id, data) → updateDoc
// listLaptops() → getDocs with orderBy('created_at', 'desc')
// searchLaptops(query) → Firestore collection queries
```

3. Photo upload in `lib/storage.ts`:
```typescript
// uploadPhoto(laptopId, file) → uploadBytes to photos/{laptopId}/{timestamp}.jpg
// getPhotoURL(path) → getDownloadURL
// deletePhoto(path) → deleteObject
```

4. Deploy Firebase Function for USB endpoint.

### Pitfalls
- Firestore `updateDoc` merges by default — test update vs set with merge
- Firebase Storage CORS — configure in Firebase Console for web client access
- Cold start on Firebase Functions — first call after deploy takes 2-3s

---

## Skill 3: Build Grading + Pricing Engine

When implementing `lib/grading.ts` and `lib/pricing.ts`.

### Grading Logic (lib/grading.ts)

```typescript
export function calculateGrade(laptop: LaptopData): Grade {
  const battery = laptop.battery?.health_pct ?? 0;
  const ssd = laptop.storage?.health_pct ?? 0;
  const bodyAvg = averageBodyScore(laptop.body_condition);
  const anyTestFailed = Object.values(laptop.tests).some(t => t.status === "fail");

  if (battery >= 85 && ssd >= 95 && bodyAvg >= 90 && !anyTestFailed) return "A+";
  if (battery >= 75 && ssd >= 80 && bodyAvg >= 70) return "A";
  if (battery >= 60 && ssd >= 60 && bodyAvg >= 50) return "B";
  if (battery >= 40 && ssd >= 40) return "C";
  return "D";
}

function averageBodyScore(condition: BodyCondition): number {
  const scores = Object.values(condition).map(v => 
    ({ excellent: 100, good: 75, fair: 50, damaged: 0 }[v] ?? 0)
  );
  return scores.reduce((a, b) => a + b, 0) / scores.length;
}
```

### Pricing Logic (lib/pricing.ts)

```typescript
const MULTIPLIERS = { "A+": 1.45, "A": 1.35, "B": 1.20, "C": 1.05, "D": 0.85 };

export function calculateSellPrice(buyPrice: number, grade: Grade): number {
  return Math.round(buyPrice * MULTIPLIERS[grade]);
}
```

### Pitfalls
- Handle null/undefined fields gracefully — a new laptop with no body condition data should grade as D, not crash
- Pricing must handle zero buy_price → show "Enter buying price" not "0 THB"
- Test with edge values: battery=40, ssd=39 (should be D not C)

---

## Skill 4: Build Gamification System

When implementing `lib/gamification.ts` and gamification UI components.

### Gamification Logic (lib/gamification.ts)

```typescript
const LEVELS = [
  { level: 1, title: "Technician Trainee", xpRequired: 0, icon: "wrench" },
  { level: 2, title: "Junior Inspector",   xpRequired: 100, icon: "binoculars" },
  { level: 3, title: "Inspector",           xpRequired: 250, icon: "clipboard-check" },
  { level: 4, title: "Senior Inspector",    xpRequired: 500, icon: "badge-check" },
  { level: 5, title: "Lead Grader",         xpRequired: 1000, icon: "shield" },
  { level: 6, title: "Master Grader",       xpRequired: 2000, icon: "crown" },
  { level: 7, title: "Spec Hunter Legend",  xpRequired: 5000, icon: "swords" },
];
```

Points: +10/laptop, +25/A+ grade, +15/A, +5 standard pass, -5 skipped test.
Badges are stored as `badges_earned: string[]` on the laptop document.
Streak tracked via `last_activity_date` — compared to today's date.

### Pitfalls
- Streak resets at midnight — use date-only comparison, not datetime
- Badges should be checked on every laptop completion, not just on grade
- XP should accumulate from a separate document (or aggregate from laptops) — don't store total XP on each laptop

---

## Skill 5: Code Review Before Commit

Run before every git commit. Must not be skipped.

### Review Workflow

```
1. git add <files>
2. claude -p 'Review staged changes for [specific concerns]. 
   Check: mobile 375px, dark theme, Lucide icons, edge cases, 
   gamification balance, Firestore safety. List issues.'
3. If issues → claude -p 'Fix the following: [issue list]'
4. git diff → verify fixes
5. git commit
```

---

## Skill 6: Deploy to Firebase

When deploying the web app or Firebase Functions.

### Steps

```bash
# Deploy web app (Next.js build + Firebase Hosting)
npm run build && firebase deploy --only hosting

# Deploy Firebase Function (USB upload endpoint)
firebase deploy --only functions

# Deploy security rules
firebase deploy --only firestore,storage

# Full deploy
firebase deploy
```

### Pre-deploy Checklist
1. Build succeeds: `npm run build`
2. All 8 tests work on real hardware
3. Photos upload to Firebase Storage
4. Grade/pricing correct
5. PWA installs on mobile
6. Review Agent ran and all issues fixed

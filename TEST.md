# Test Plan: Spec Hunter — Web App

## Test Levels

| Level | What | When | Tool |
|-------|------|------|------|
| Unit | Individual functions (grading, pricing, gamification) | After each backend function | vitest |
| Integration | Firestore CRUD + Firebase Function | After lib/ is wired | Manual + firebase emulator |
| E2E | Full user flow (create → test → grade → price) | Before major push | Manual browser |
| Edge Cases | Empty states, errors, network fail | Before production | Manual |

## Test Cases

### Feature: Manual Entry Form

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Happy path — fill all fields | Fill brand, model, CPU, RAM, storage, battery, display, network → Submit | Document created in Firestore, redirect to detail page | ☐ |
| TC2 | Minimal data — only serial + brand | Enter serial + brand only → Submit | Record created with defaults/null for other fields | ☐ |
| TC3 | Paste JSON — valid payload | Paste valid JSON → Submit | All fields parsed, document created | ☐ |
| TC4 | Paste JSON — malformed | Paste garbage text → Submit | Error message displayed, no document created | ☐ |
| TC5 | Duplicate serial | Submit same serial twice | Warning shown, duplicate allowed or blocked? | ☐ |
| TC6 | Empty form | Click Submit with empty form | Validation errors on required fields | ☐ |

### Feature: Keyboard Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | All keys pressed | Press every key on virtual keyboard | All keys turn green, auto-PASS | ☐ |
| TC2 | Partial keys | Press only alphabetic keys | Those keys green, others gray, status = pending | ☐ |
| TC3 | Failed keys on FAIL | Press only 3 keys then fail | failed_keys[] recorded with unpressed keys | ☐ |

### Feature: LCD Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Full cycle, PASS | Click next through all colors, click PASS | Each color goes fullscreen, PASS recorded | ☐ |
| TC2 | Full cycle, FAIL with defect | Click PASS through colors, pick "dead_pixel", click FAIL | FAIL + issue: "dead_pixel" recorded | ☐ |
| TC3 | Exit fullscreen mid-test | Press Escape | Returns to test page, current color state preserved | ☐ |

### Feature: Speaker Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Left + Right PASS | Play left → click PASS → play right → click PASS | Both channels recorded as pass | ☐ |
| TC2 | Left fail, Right pass | Play left → FAIL → play right → PASS | Speaker test=FAIL, left_pass=false, right_pass=true | ☐ |

### Feature: Camera Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Camera opens, capture | Click Start → allow camera → click Capture | Thumbnail shown, auto-PASS | ☐ |
| TC2 | Camera denied | Click Start → deny permission | Graceful error, no crash | ☐ |
| TC3 | No camera on device | Run on desktop with no webcam | Message: "No camera detected" + manual PASS/FAIL | ☐ |

### Feature: Microphone Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Record + playback | Click Record → speak → click Stop → Playback | Audio plays back, manual PASS/FAIL | ☐ |
| TC2 | Mic denied | Click Record → deny permission | Graceful error, manual FAIL | ☐ |

### Feature: Touchpad Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Move + click | Move cursor across screen → click | Both detected, auto-PASS | ☐ |
| TC2 | Click only (no movement) | Click without moving | Movement not detected, still pending | ☐ |
| TC3 | Movement only (no click) | Move cursor without clicking | Click not detected, still pending | ☐ |

### Feature: USB Port Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | PASS | Plug USB → click "Detected" → PASS | USB test = PASS | ☐ |
| TC2 | FAIL | Click FAIL without plugging | USB test = FAIL | ☐ |

### Feature: WiFi Test

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Online | Run test on internet-connected device | PASS with latency_ms shown | ☐ |
| TC2 | Offline | Run test with airplane mode on | FAIL recorded | ☐ |

### Feature: Photo Upload

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Upload 1 photo | Tap upload → select photo | Thumbnail appears, photo_count=1 | ☐ |
| TC2 | Upload 10 photos | Upload until limit | Gallery shows all 10, 11th blocked | ☐ |
| TC3 | Delete a photo | Click delete on uploaded photo | Photo removed from Storage + gallery | ☐ |
| TC4 | Phone camera capture | Tap upload on phone | Camera opens, capture uploads directly | ☐ |

### Feature: Grading Engine

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | A+ grade | battery=90, ssd=96, body=all excellent, no test failures | Grade = A+ | ☐ |
| TC2 | A grade | battery=80, ssd=85, body=good, no failures | Grade = A | ☐ |
| TC3 | B grade | battery=65, ssd=70, body=fair, no failures | Grade = B | ☐ |
| TC4 | C grade | battery=45, ssd=50, body=fair | Grade = C | ☐ |
| TC5 | D grade | battery=30, ssd=30 | Grade = D | ☐ |
| TC6 | Test failure downgrade | battery=90, ssd=96, but LCD test = fail | Grade = A (not A+) | ☐ |

### Feature: Pricing

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | A grade calculation | buy_price=6500, grade=A | suggested_price=8775 (6500×1.35) | ☐ |
| TC2 | No buy_price entered | grade=A, buy_price=null | "Enter buying price to see suggestion" message | ☐ |

### Feature: Gamification

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Points on laptop processed | Complete a laptop (all tests + grade) | +10 points added to XP | ☐ |
| TC2 | Level up | Earn 100 XP as technician trainee | Level 2 "Junior Inspector" unlocked | ☐ |
| TC3 | Streak increment | Process laptops on consecutive days | Streak counter increases each day | ☐ |
| TC4 | Streak reset | Skip a day | Streak resets to 0 | ☐ |
| TC5 | Badge earned | First A+ grade | A+ Master badge appears in profile | ☐ |

### Feature: Dashboard

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | Today's stats | Process laptops, check dashboard | processed/pending/passed/rejected counts match | ☐ |
| TC2 | Empty state | No laptops processed today | "No laptops processed today" message + 0 counts | ☐ |
| TC3 | Streak + level display | Has streak and level | Widgets show current streak + level + XP bar | ☐ |

### Feature: Table View

| # | Test Case | Steps | Expected | Status |
|---|-----------|-------|----------|--------|
| TC1 | List all laptops | Navigate to /laptops | All records displayed in table | ☐ |
| TC2 | Search by serial/brand | Type "ThinkPad" in search | Filtered results | ☐ |
| TC3 | Filter by status | Select "pending" filter | Only pending laptops shown | ☐ |
| TC4 | Sort by column | Click column header (e.g., "Grade") | Rows sorted ascending/descending | ☐ |
| TC5 | Empty table | No records | "No laptops yet. Create your first one." message | ☐ |
| TC6 | Loading state | Slow Firestore read | Skeleton/spinner shown, not blank page | ☐ |

## Acceptance Criteria Checklist

| # | Category | Check | How to verify |
|---|----------|-------|---------------|
| 1 | ✅ Happy path | Full laptop lifecycle works | Create → all tests → photos → grade → price |
| 2 | ❌ Error case | API fails, permission denied | Console shows error, user sees friendly message |
| 3 | ⚠️ Edge case | Empty fields, extreme values, rapid clicks | Graceful handling, no crashes |
| 4 | 📱 Responsive | 375px, 768px, 1280px | Layouts adapt, all interactive elements tappable |
| 5 | 🎨 Theme | Dark mode renders | bg #121317, surface #1a1b1f, accent #2e96ff everywhere |
| 6 | 🚫 Empty state | No data yet | "No laptops yet" message, not blank page |
| 7 | 🔄 Loading state | During Firestore reads | Spinner/skeleton, not frozen UI |
| 8 | 📷 Camera/mic | Permission flow | Browser prompts, graceful denial handling |

# Spec: Spec Hunter — Web App

## 1. The Gist

A mobile-first PWA for wholesale laptop intake — operators create hardware records, run 8 interactive tests in the browser, upload photos from phone, get auto-grades and pricing suggestions. Built for JM505 shop.

## 2. The Story

A batch of 20 used ThinkPads arrives from a Bangkok wholesaler. Bhone needs to inspect each one: check the screen, keyboard, speakers, camera, battery health. Old workflow: sticky notes on each laptop, manual spreadsheet, photos on phone with no link to records. New workflow: open the PWA on his phone, test keyboard by actually pressing keys (it tracks them), cycle through fullscreen LCD colors, snap body photos that upload directly, and the system suggests a grade + resale price automatically.

## 3. The "Why"

Wholesale laptop grading is repetitive, error-prone, and documentation never happens when you're processing 20+ units/day. Without a system, you lose track of which laptop has what defect, what you paid, and what to sell it for. This removes all the manual paperwork and gives instant, consistent grading.

## 4. The "Why Not" (Anti-Goals) 🛑

- **No auth** — Single operator, internal tool only. No login page, no user management.
- **No USB tool here** — USB hardware collector is a separate repo. This web app accepts manual entry or pasted JSON.
- **No PostgreSQL** — Firestore only. No migrations, no SQL.
- **No QR codes** — Row-based workflow is fine for single operator.
- **No ML pricing** — Formula-based only. ML comes later with 50+ sales data points.
- **No offline mode** — Requires internet for Firestore reads/writes. PWA install is for home-screen shortcut only.
- **No export/CSV** — Not in MVP. Future feature.
- **No multi-language** — English UI only.
- **NO emoji** — Use Lucide icons exclusively for all UI elements including gamification.

## 5. Technical Spec

### Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router) |
| Styling | Tailwind CSS |
| Icons | Lucide React (NO emoji) |
| Database | Firestore (NoSQL) |
| File Storage | Firebase Storage |
| Backend Logic | Firebase Functions (Node.js) |
| Hosting | Firebase Hosting |
| PWA | next-pwa or next.config PWA config |
| Auth | None (single operator) |

### Firestore Schema

```
laptops/{laptopId}
├── identity: { brand, model, serial_number, bios_version, manufacture_date, motherboard_model, uuid, asset_tag }
├── cpu: { name, generation, cores, threads, base_clock, turbo_clock }
├── ram: { total_gb, slot_count, used_slots, speed, ddr_type, manufacturer }
├── storage: { model, capacity_gb, interface, health_pct, power_on_hours, power_cycles, total_bytes_written, remaining_life_pct, temperature, bad_sectors, smart_status }
├── battery: { design_capacity, current_capacity, cycle_count, health_pct, manufacturer, serial }
├── display: { resolution, refresh_rate, manufacturer, screen_size_inch, model }
├── network: { wifi_card, bluetooth, mac_address, lan_adapter }
├── body_condition: { top_cover, bottom_cover, keyboard, palmrest, screen, left_side, right_side, front, back }
│   // values: "excellent" | "good" | "fair" | "damaged"
├── accessories: string[]  // ["adapter", "box", ...]
├── tests: {
│     keyboard: { status: "pass"|"fail"|"pending", failed_keys: string[] },
│     speaker:  { status: "pass"|"fail"|"pending", left_pass?: boolean, right_pass?: boolean },
│     camera:   { status: "pass"|"fail"|"pending", thumbnail_url?: string },
│     lcd:      { status: "pass"|"fail"|"pending", issue?: "dead_pixel"|"bright_pixel"|"bleeding"|"lines"|"burn_in"|"pressure_mark"|null },
│     touchpad: { status: "pass"|"fail"|"pending" },
│     usb:      { status: "pass"|"fail"|"pending" },
│     microphone: { status: "pass"|"fail"|"pending" },
│     wifi:     { status: "pass"|"fail"|"pending", latency_ms?: number }
│   }
├── grade: "A+" | "A" | "B" | "C" | "D" | null
├── buy_price: number | null
├── sell_price: number | null
├── suggested_price: number | null
├── price_note: string | null
├── status: "pending" | "testing" | "passed" | "rejected"
├── photo_count: number
├── operator_notes: string
├── created_at: timestamp
├── updated_at: timestamp
├── points_earned: number       // gamification
└── badges_earned: string[]     // gamification
```

### Pages

| Route | Content |
|-------|---------|
| `/` | Dashboard — today's stats (processed, pending, passed, rejected), streak, level, badges |
| `/laptops` | Table view — search, filter by brand/status/grade, sort |
| `/laptops/new` | Manual entry form + paste-JSON option |
| `/laptops/[id]` | Detail page — full specs, test status grid, photo gallery, grade badge |
| `/laptops/[id]/test` | Test suite — step through all 8 tests |
| `/laptops/[id]/grade` | Grade result + pricing + body condition form |
| `/profile` | Operator stats — XP, level, streak, badge collection |

### Interactive Tests (Browser APIs)

| Test | API | Pass Condition |
|------|-----|---------------|
| Keyboard | `keydown` + virtual keyboard SVG | Auto-PASS when all keys pressed |
| LCD | Fullscreen API + `<div>` color fills | Manual PASS/FAIL + defect picker |
| Speaker | Web Audio API (OscillatorNode) | Manual PASS/FAIL (L + R channels) |
| Camera | `getUserMedia()` + canvas snapshot | Auto-PASS on successful capture |
| Microphone | `MediaRecorder` → playback | Manual PASS/FAIL |
| Touchpad | `pointermove` + `click` events | Auto-PASS if pointer moved + clicked |
| USB Port | Manual button press | Manual PASS/FAIL |
| WiFi | `fetch()` ping with timeout | Auto-PASS if reachable |

### Grading Engine

```
A+ = battery>=85% AND ssd>=95% AND bodyAvg>=90 AND no test failures
A  = battery>=75% AND ssd>=80% AND bodyAvg>=70
B  = battery>=60% AND ssd>=60% AND bodyAvg>=50
C  = battery>=40% AND ssd>=40%
D  = below C thresholds (needs repair)
```

Body part scoring: excellent=100, good=75, fair=50, damaged=0. Averaged across all 9 parts.

### Pricing Formula

```
suggested_price = buy_price × multiplier

A+ = 1.45x    A = 1.35x    B = 1.20x    C = 1.05x    D = 0.85x
```

### Gamification System

| Level | Title | XP Required | Icon |
|-------|-------|-------------|------|
| 1 | Technician Trainee | 0 | wrench |
| 2 | Junior Inspector | 100 | binoculars |
| 3 | Inspector | 250 | clipboard-check |
| 4 | Senior Inspector | 500 | badge-check |
| 5 | Lead Grader | 1000 | shield |
| 6 | Master Grader | 2000 | crown |
| 7 | Spec Hunter Legend | 5000 | swords |

Points: +10/laptop, +25/A+ grade, +15/A, +5 standard, -5 skipped test.

### Firebase Functions

```
POST /uploadLaptop
  Description: Accept hardware JSON from USB boot tool
  Headers: x-api-key: <shared-secret>
  Body: { identity, cpu, ram, storage, battery, display, network, camera }
  Response: { laptopId: string, url: string }
```

## 6. Definition of Done

| # | Feature | How to Verify |
|---|---------|---------------|
| 1 | Firebase project created, Firestore + Storage + Functions enabled | `firebase deploy` succeeds |
| 2 | Next.js app scaffolded with Tailwind + Lucide + Firebase SDK | `npm run dev` shows blank page with no errors |
| 3 | Firestore types (TypeScript) match schema | Types compile without errors |
| 4 | Manual entry form — all hardware fields | Submit creates document in Firestore |
| 5 | Paste-JSON option — raw JSON creates record | Paste valid JSON → record appears |
| 6 | Laptop table view — search, filter, sort | Search by brand, filter by status, sort by date |
| 7 | Laptop detail page — all specs displayed | Click row → see full specs, test status, photos |
| 8 | Photo upload — phone camera capture → Firebase Storage → gallery | Upload 10 photos, see thumbnails |
| 9 | Keyboard test — SVG layout, key tracking, auto-PASS | Press all keys → PASS recorded |
| 10 | LCD test — fullscreen colors + defect picker | Cycle colors, pick defect, submit PASS/FAIL |
| 11 | Speaker test — play L/R channels | Hear tones, click PASS/FAIL |
| 12 | Camera test — live preview + capture | Camera opens, capture saves thumbnail |
| 13 | Microphone test — record 3s + playback | Record → hear playback → PASS/FAIL |
| 14 | Touchpad test — pointer + click detection | Move cursor + click → auto-PASS |
| 15 | USB port test — manual button | Click Detected → PASS/FAIL |
| 16 | WiFi test — fetch ping | Internet reachable → PASS with latency |
| 17 | Body condition form — 9 parts × 4 options | All fields save to Firestore |
| 18 | Grading engine — auto-calculates on save | Grade badge updates correctly |
| 19 | Pricing suggestion — formula based on grade + buy_price | Suggested price appears with calculation |
| 20 | Dashboard — today's stats | Processed/pending/passed/rejected show correct numbers |
| 21 | Gamification — points, levels, streak, badges | +10 per laptop, level progress bar, streak counter |
| 22 | Profile page — XP, level, badge collection | All gamification data displayed |
| 23 | PWA — install prompt on mobile | "Add to Home Screen" appears |
| 24 | Dark theme throughout — bg #121317, surface #1a1b1f, accent #2e96ff | All pages render in dark theme |
| 25 | Firebase Function — uploadLaptop endpoint | POST sample JSON → returns laptopId |
| 26 | Mobile-responsive — all pages work at 375px | Resize browser → layouts adapt |
| 27 | Code Review — Review Agent run before commit | git add → load Review Agent → issues fixed or "✅ No issues" → commit |
| 28 | Fix Agent — issues from review are fixed | Review Agent issues resolved, verified with git diff |

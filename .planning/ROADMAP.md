# Spec Hunter — Roadmap

## Milestones

### Phase 1 — Web App Core (Current)
**Goal:** Working PWA with manual entry, all 8 interactive tests, photo upload, grading, pricing, gamification.

| Step | What | Depends On |
|------|------|------------|
| 1.1 | Firebase project + Next.js scaffold | Nothing |
| 1.2 | Firestore schema + types | 1.1 |
| 1.3 | Manual entry form + paste JSON | 1.2 |
| 1.4 | Laptop table + detail page | 1.3 |
| 1.5 | Keyboard + LCD + Speaker tests | 1.4 |
| 1.6 | Camera + Microphone + Touchpad tests | 1.4 |
| 1.7 | USB + WiFi tests | 1.4 |
| 1.8 | Photo upload + gallery | 1.2 |
| 1.9 | Body condition + Grading engine | 1.4 |
| 1.10 | Pricing suggestion | 1.9 |
| 1.11 | Dashboard + profile | 1.10 |
| 1.12 | Gamification (points, levels, badges, streak) | 1.11 |
| 1.13 | PWA setup + deploy to Firebase | 1.12 |

### Phase 2 — USB Boot Collector
**Goal:** Headless Alpine ISO that collects hardware and uploads to API.

See `../spec-hunter-usb/` — separate repo.

### Phase 3 — Production
**Goal:** QR stickers, marketplace pricing, profit analytics, multi-operator.

No timeline yet. Dependent on real usage feedback.

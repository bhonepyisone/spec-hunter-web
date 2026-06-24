# Spec Hunter — Project Context

## Overview

Mobile-first PWA for wholesale laptop intake and grading. Operators create hardware records (manually or via USB upload), run 8 interactive tests in the browser, upload photos from phone, get auto-grades and pricing suggestions.

**For:** JM505 shop (single operator, internal tool)
**Target users:** Bhone (shop owner) processing 20+ laptops/day

## Key Constraints

- Single operator — no auth/login
- Firebase only: Firestore + Storage + Functions + Hosting
- NO emoji in UI — Lucide icons exclusively
- Dark theme: bg #020617, surface #1A1E2F, accent #22C55E
- Battery thresholds: A+≥85, A≥75, B≥60, C≥40, D<40
- Pricing: buy_price × grade_multiplier
- All interactive tests in browser using browser APIs
- USB tool is SEPARATE repo — API contract only

## Stack

- Next.js 14+ (App Router), Tailwind CSS, Firebase
- Design system: UI-UX Pro Max Data-Dense Dashboard
- Fonts: Fira Code (headings), Fira Sans (body)
- Icons: Lucide React

## External Dependencies

- USB tool (separate repo) → POST JSON to Firebase Function
- No external APIs needed for MVP

## Design System Location

`design-system/spec-hunter/MASTER.md`

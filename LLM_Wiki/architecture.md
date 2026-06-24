# Spec Hunter Web — Architecture

## System Design

```
Phone/PWA Browser                  Firebase
      │                               │
      ├── Firestore SDK ──────────────▶ Firestore
      ├── Firebase Storage ───────────▶ Storage (photos)
      │                               │
USB Tool                              │
      │                               │
      └── POST JSON + API Key ────────▶ Firebase Function
                                          │
                                          └──▶ Firestore
```

## Data Flow

### Laptop Lifecycle

```
1. CREATE: Manual entry OR paste JSON OR USB upload
       → Firestore document created, status="pending"
       
2. TESTING: Operator runs tests one-by-one
       → Each test updates Firestore {tests.{name}.status}
       → After all tests done, status="testing" or can be set directly
       
3. GRADEING: Operator fills body condition + hits Grade button
       → Grading engine calculates grade
       → Pricing formula suggests sell price
       → status="passed" or "rejected"
       
4. PHOTOS: Uploaded anytime during process
       → Stored in Firebase Storage: photos/{laptopId}/{timestamp}.jpg
```

### Gamification Data Flow

```
Every laptop completion:
  1. Points calculated (base + grade_bonus - penalties)
  2. XP incremented in a gamification document or laptop.points_earned
  3. Level checked (XP thresholds)
  4. Streak checked (last_activity_date vs today)
  5. Badges checked (achievement conditions)
  6. All updates written to Firestore
```

## Firestore Security Rules (Draft)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow read from web client (single operator, no auth)
    match /laptops/{document} {
      allow read: if true;
      allow write: if request.auth != null || request.resource.data.api_key == 'usb-upload-key';
    }
  }
}
```

Note: For MVP without Firebase Auth, use Firebase Function for USB writes and allow client-side writes from web app (internal tool assumption).

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| No SSR/SSG for laptop pages | Static pages are fine — data loads client-side from Firestore. No SEO needed for internal tool. |
| PWA but no offline | Firestore requires internet. PWA is just for home-screen shortcut. |
| Single Firestore doc per laptop | Tests and grade are fields inside the laptop doc, not separate collections. Simpler reads. |
| Firebase Function for USB upload | Avoids exposing Firestore write access to external clients. API key as simple gate. |

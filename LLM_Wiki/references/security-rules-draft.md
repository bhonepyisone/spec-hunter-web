# Firestore + Storage Security Rules — Draft

> Draft rules for Spec Hunter MVP. These are applied at `firebase deploy --only firestore,storage`.
> Review before first deploy — adjust based on actual Firebase project setup.

---

## Firestore Rules (firestore.rules)

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ──── Security Model ────
    // MVP: Single operator, no Firebase Auth.
    // - Web app reads/writes directly from client-side SDK
    // - USB uploads via Firebase Function (admin SDK bypasses rules)
    // - No sensitive data — laptop specs + photos only
    //
    // Future: Add auth check when multi-operator is needed
    //   allow write: if request.auth != null && request.auth.uid == 'operator-uid';

    match /laptops/{laptopId} {

      // Read: Anyone can read (single operator, private URL)
      allow read: if true;

      // Write: Web client (no auth) OR Firebase Function (admin bypass)
      // For MVP: allow write from any client
      allow create: if true;
      allow update: if true;
      allow delete: if false;  // No delete from client — use Firebase Console or Function

      // ──── Data Validation (optional but recommended) ────
      // Uncomment to enforce required fields on create:
      // allow create: if request.resource.data.identity.brand is string
      //              && request.resource.data.created_at is number;

      // Prevent overwriting certain fields
      allow update: if request.resource.data.created_at == resource.data.created_at;
    }

    // ──── Gamification Data ────
    // Store operator stats in a separate document
    match /stats/{doc} {
      allow read: if true;
      allow write: if true;
    }
  }
}
```

### Key Decisions

| Decision | Why |
|----------|-----|
| `allow read: if true` | Single operator behind private URL. No sensitive data. |
| `allow write: if true` (web client) | No auth in MVP. Client SDK writes directly. |
| `allow delete: if false` | Prevent accidental data loss. Delete via Firebase Console if needed. |
| Firebase Function bypasses rules | Admin SDK has full access regardless of rules. |
| Future: auth check | Add `request.auth.uid == 'operator-uid'` when adding Firebase Auth. |

### Multi-Operator Future (Phase 3)

```javascript
// Replace the write rules above with:
match /laptops/{laptopId} {
  allow read: if true;
  allow create, update: if request.auth != null
                        && request.auth.uid == resource.data.assigned_to;
  allow delete: if false;
}
```

---

## Storage Rules (storage.rules)

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {

    // Photos directory: read all, write by web client
    match /photos/{laptopId}/{fileName} {
      allow read: if true;
      allow write: if true
        && request.resource.size < 10 * 1024 * 1024  // Max 10MB per file
        && fileName.matches('.*\\.(jpg|jpeg|png|webp)$');  // Images only
      allow delete: if false;
    }

    // Uploads directory: only Firebase Function can write
    match /uploads/{allPaths=**} {
      allow read: if false;
      allow write: if false;  // Not used in MVP — Function uses admin SDK
    }

    // App files (manifest, icons, etc.)
    match /{allPaths=**} {
      allow read: if true;
      allow write: if false;
    }
  }
}
```

### Key Decisions

| Decision | Why |
|----------|-----|
| Max 10MB per photo | Phone photos are typically 2-5MB. 10MB covers all cases. |
| Only jpg/jpeg/png/webp | Block malicious uploads. HEIC/heif not supported by Firebase SDK. |
| `allow delete: if false` | Prevent accidental deletion. Remove via Firebase Console. |

---

## Photo Upload Path Convention

```
photos/{laptopId}/{timestamp}_{index}.jpg

Example:
photos/abc123xyz/1719504000000_1.jpg
photos/abc123xyz/1719504000000_2.jpg
```

- `timestamp` = `Date.now()` in milliseconds — ensures unique filenames
- `index` = 1-10, sequential per batch
- No spaces or special chars in filename
- All lowercase for consistency

---

## Deployment

```bash
# Apply rules
firebase deploy --only firestore
firebase deploy --only storage

# Verify
# Go to Firebase Console → Firestore → Rules tab → check "Last deployed"
# Go to Firebase Console → Storage → Rules tab → check "Last deployed"
```

## Testing

| Test | Steps | Expected |
|------|-------|----------|
| Read from web | Open app → laptops load | Firestore read succeeds |
| Write from web | Create laptop → form submit | Document created in Firestore |
| Photo upload | Upload 5MB JPEG | File saved in Storage |
| Photo too large | Upload 15MB file | Rule denies write, 403 error |
| Wrong file type | Upload .exe file | Rule denies write, 403 error |
| Delete from web | Try to delete a laptop doc | Firestore denies (delete: false) |

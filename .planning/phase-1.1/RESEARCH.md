# Phase 1.1 — Research: Firebase + Next.js Setup

## Firebase Setup

### Creating a Firebase Project
1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click "Create a project" → name `spec-hunter` → accept terms → create
3. Enable **Firestore**: Native mode, location `asia-southeast1`
4. Enable **Storage**: Default rules, location `asia-southeast1`
5. Enable **Functions**: Node.js 20, pay-as-you-go billing
6. Register **Web app** in Project Settings → "Add app" → Web → copy config

### Firebase CLI
```bash
npm install -g firebase-tools
firebase login
firebase init
# Select: Firestore, Functions, Storage, Hosting
# Use existing project → spec-hunter
```

## Next.js + Firebase Integration

### Key Packages
| Package | Purpose |
|---------|---------|
| `firebase` | Client-side Firestore + Storage SDK |
| `firebase-admin` | Server-side (Functions) admin access |
| `lucide-react` | Icons (NO emoji) |
| `next-pwa` | PWA support (or built-in Next.js config) |

### Firebase Config Pattern
```typescript
// lib/firebase.ts
import { initializeApp, getApps } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';
import { getStorage } from 'firebase/storage';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];
export const db = getFirestore(app);
export const storage = getStorage(app);
```

### Tailwind Config with Design Tokens
```typescript
// tailwind.config.ts
export default {
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        background: '#020617',
        surface: '#1A1E2F',
        primary: '#0F172A',
        accent: '#22C55E',
        foreground: '#F8FAFC',
        border: '#334155',
        destructive: '#EF4444',
      },
      fontFamily: {
        heading: ['Fira Code', 'monospace'],
        body: ['Fira Sans', 'sans-serif'],
      },
    },
  },
};
```

### Google Fonts Import
```tsx
// app/layout.tsx
import '@fontsource/fira-code';
import '@fontsource/fira-sans';
// Or use CSS @import:
// @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500;600;700&family=Fira+Sans:wght@300;400;500;600;700&display=swap');
```

## Potential Issues
1. **Firebase Functions cold start** — First call after deploy is slow (2-3s). Acceptable for internal tool.
2. **CORS on Storage** — Must configure in Firebase Console for web client uploads.
3. **`asia-southeast1` limits** — Some Firebase features deploy slower than `us-central1`.
4. **Next.js 15** — If available, check compatibility with firebase SDK before using.

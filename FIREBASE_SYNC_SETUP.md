# Firebase Sync Setup

This project now supports cross-device progress sync using Firebase Auth + Firestore.

## 1) Create Firebase project

1. Go to Firebase Console and create a project.
2. Add a **Web App**.
3. Copy the web config values (`apiKey`, `authDomain`, `projectId`, etc.).

## 2) Enable services

1. **Authentication**:
   - Enable **Google** provider (recommended).
2. **Firestore Database**:
   - Create database in production mode.
   - Pick region close to your users.

## 3) Configure app

Choose one method:

- **Method A (recommended)**: put config directly in `index.html` inside `FIREBASE_CONFIG`.
- **Method B**: create `firebase-config.js` from `firebase-config.example.js` and include it before the main inline script.

The app automatically falls back to local mode if config is missing.

## 4) Apply Firestore rules

Use the rules in `firestore.rules`:

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

## 5) Data model

Collection: `users`  
Document ID: Firebase `uid`

Fields:

- `theme`
- `statuses`
- `pinned`
- `lastOpened`
- `activity`
- `updatedAtClient`
- `updatedAt` (server timestamp)
- `schemaVersion`

## 6) Behavior summary

- Local state is always available (`localStorage` fallback).
- On sign-in, cloud state is loaded and merged with local.
- Merge strategy:
  - `updatedAtClient` newer state wins overall.
  - `activity` combines and trims.
  - `statuses`, `lastOpened` merge keys.
- Auto-sync writes are debounced.
- Realtime listener updates state from other devices.

## 7) Validation checklist

1. Sign in on Device A and mark a problem `Done`.
2. Open site on Device B (same account) and sign in.
3. Confirm status appears on Device B.
4. Pin/unpin and verify realtime reflection across devices.
5. Go offline on Device A, make a change, go online, verify sync resumes.
6. Confirm user isolation by testing another account.

## 8) Troubleshooting

- If UI says `Local only`:
  - Firebase config is missing/placeholder.
  - SDK script blocked by network/cors.
- If `Auth error`:
  - Google provider not enabled.
  - Unauthorized domain not added.
- If `Sync error`:
  - Firestore rules not deployed correctly.
  - Firestore not initialized.

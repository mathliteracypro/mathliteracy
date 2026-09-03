# 11. Authentication & Onboarding Backend Documentation

This document describes the sign-in mechanisms (Google OAuth & Student ID authentication), user creation hooks, and onboarding state synchronization.

---

## 1. Document Schemas

### Onboarding Progress State (`onboarding/{uid}`)

```json
{
  "uid": "google-user-123",
  "currentStepIndex": 4,
  "isComplete": true,
  "skipped": false,
  "startedAt": "2026-08-31T11:00:00.000Z",
  "completedAt": "2026-08-31T11:05:00.000Z",
  "stepData": {
    "welcome": { "agreedToTerms": true },
    "profile": { "role": "student", "phone": "+27 82 000 0000" },
    "grade": { "grade": 11 },
    "subjects": { "interests": ["Finance", "Tariffs", "Maps"] },
    "goals": { "targetPercentage": 85 }
  }
}
```

---

## 2. Authentication Flow & Account Auto-Provisioning

When a user signs in via Google OAuth:
1. Check if document exists at `users/{uid}`.
2. If missing, auto-create student record with:
   - `role: 'student'`
   - `status: 'active'`
   - `onboardingCompleted: false`
   - `studentId: generateStudentId()`
3. Initialize onboarding document at `onboarding/{uid}`.

```typescript
import { doc, getDoc, setDoc } from 'firebase/firestore';

export async function syncUserOnAuth(user: { uid: string; email: string; displayName: string; photoURL: string }) {
  const userRef = doc(db, 'users', user.uid);
  const snap = await getDoc(userRef);

  if (!snap.exists()) {
    const newUser = {
      uid: user.uid,
      email: user.email,
      displayName: user.displayName,
      photoURL: user.photoURL,
      role: 'student',
      status: 'active',
      studentId: `MLPRO${Date.now().toString().slice(-6)}${Math.floor(1000 + Math.random() * 9000)}`,
      onboardingCompleted: false,
      createdAt: new Date().toISOString()
    };
    await setDoc(userRef, newUser);
    return newUser;
  }
  return snap.data();
}
```

---

## 3. Onboarding Completion Redirection Hook

When `onboardingService.completeOnboarding(userId)` is invoked:
1. `onboarding/{uid}` is updated with `isComplete: true` and `completedAt: ISO string`.
2. `users/{uid}` is updated with `onboardingCompleted: true`.
3. Application sets view state to `'dashboard'` and navigates to `/dashboard`.

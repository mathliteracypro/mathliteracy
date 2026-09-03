# 04. User & Role Management Backend Documentation

This document describes the backend user directory, role-based access control (RBAC), user account status tracking (active, suspended, banned), and unique Student ID generation algorithms.

---

## 1. Firestore Collection Schema (`users/{uid}`)

```json
{
  "uid": "google-oauth-uid-998877",
  "studentId": "MLPRO26083156801",
  "displayName": "Sipho Nkosi",
  "email": "sipho.nkosi@example.co.za",
  "photoURL": "https://lh3.googleusercontent.com/...",
  "role": "student",
  "status": "active",
  "grade": 11,
  "schoolName": "Polokwane High School",
  "province": "Limpopo",
  "parentEmail": "parent.nkosi@example.co.za",
  "onboardingCompleted": true,
  "stats": {
    "topicsCompleted": 3,
    "tasksSubmitted": 12,
    "practiceScoreAvg": 82.5,
    "currentStreakDays": 7,
    "totalTimeSpentMinutes": 640
  },
  "lastLoginAt": "2026-08-31T11:00:00.000Z",
  "createdAt": "2026-01-15T09:30:00.000Z",
  "updatedAt": "2026-08-31T11:00:00.000Z"
}
```

---

## 2. Admin CRUD Operations

### Fetch Users List with Filters
```typescript
import { collection, query, where, orderBy, limit, getDocs } from 'firebase/firestore';

export async function fetchUserDirectory(filters: { role?: string; status?: string; grade?: number }) {
  const usersRef = collection(db, 'users');
  let constraints: any[] = [];

  if (filters.role) constraints.push(where('role', '==', filters.role));
  if (filters.status) constraints.push(where('status', '==', filters.status));
  if (filters.grade) constraints.push(where('grade', '==', filters.grade));

  const q = query(usersRef, ...constraints, limit(100));
  const snap = await getDocs(q);
  return snap.docs.map(doc => ({ uid: doc.id, ...doc.data() }));
}
```

### Update User Role & Privileges
```typescript
import { doc, updateDoc } from 'firebase/firestore';

export async function updateUserRole(uid: string, newRole: 'admin' | 'instructor' | 'moderator' | 'student' | 'parent') {
  const userRef = doc(db, 'users', uid);
  await updateDoc(userRef, {
    role: newRole,
    updatedAt: new Date().toISOString()
  });
}
```

### Moderate Account Status (Suspend / Ban / Reactivate)
```typescript
export async function setUserAccountStatus(uid: string, status: 'active' | 'suspended' | 'banned', reason?: string) {
  const userRef = doc(db, 'users', uid);
  await updateDoc(userRef, {
    status,
    statusReason: reason || null,
    updatedAt: new Date().toISOString()
  });
}
```

---

## 3. Unique Student ID Generation Algorithm

Every learner account can receive a unique branded Student ID (`MLPRO` + Timestamp + Random digits).

```typescript
export function generateStudentId(): string {
  const prefix = 'MLPRO';
  const timestamp = Date.now().toString().slice(-6);
  const randomDigits = Math.floor(1000 + Math.random() * 9000).toString();
  return `${prefix}${timestamp}${randomDigits}`;
}
```

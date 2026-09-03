# 05. Instructors Management Backend Documentation

This document describes educator profiles, assigned topics/courses, background verification metadata, and student evaluation ratings.

---

## 1. Schema Specifications (`instructor_profiles/{instructorId}`)

```json
{
  "id": "inst-101",
  "uid": "google-uid-educator-01",
  "fullName": "Dr. Phumzile Sithole",
  "title": "Head of Mathematics Literacy",
  "email": "phumzile.sithole@mathliteracypro.co.za",
  "phone": "+27 82 555 1234",
  "avatarUrl": "https://images.unsplash.com/photo-1573496359142-b8d87734a5a2",
  "bio": "15+ years of CAPS curriculum design and national examination marking experience.",
  "qualifications": ["B.Ed Mathematics", "M.Ed Curriculum Studies (Wits)"],
  "assignedTopics": ["finance-tariffs-g11", "measurement-g12", "data-handling-g10"],
  "ratingAvg": 4.9,
  "ratingCount": 128,
  "status": "active",
  "createdAt": "2026-02-01T10:00:00.000Z"
}
```

---

## 2. Admin CRUD Operations

### Create Educator Profile
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function createInstructorProfile(profileData: any) {
  const profileRef = doc(db, 'instructor_profiles', profileData.id);
  await setDoc(profileRef, {
    ...profileData,
    createdAt: new Date().toISOString()
  });

  // Also update standard user role to instructor
  if (profileData.uid) {
    const userRef = doc(db, 'users', profileData.uid);
    await updateDoc(userRef, { role: 'instructor' });
  }
}
```

### Assign Courses to Instructor
```typescript
import { doc, updateDoc, arrayUnion } from 'firebase/firestore';

export async function assignTopicToInstructor(instructorId: string, topicId: string) {
  const profileRef = doc(db, 'instructor_profiles', instructorId);
  await updateDoc(profileRef, {
    assignedTopics: arrayUnion(topicId)
  });
}
```

### List Instructors for Directory
```typescript
import { collection, getDocs, query, where } from 'firebase/firestore';

export async function getActiveInstructors() {
  const q = query(collection(db, 'instructor_profiles'), where('status', '==', 'active'));
  const snap = await getDocs(q);
  return snap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
}
```

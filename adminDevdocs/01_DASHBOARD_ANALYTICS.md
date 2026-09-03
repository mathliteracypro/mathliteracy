# 01. Dashboard & Analytics Backend Documentation

This document covers the backend data sources, aggregation queries, and real-time listeners required for the **Dashboard Page** (`/dashboard` or `/admin/dashboard`).

---

## 1. Page Purpose & Summary

The Dashboard provides high-level system metrics, active learner tracking, pending assignment tasks, completion trends, and CAPS performance overviews for administrators and educators.

---

## 2. Firestore Collections & Queries

### A. Total Active Users Metric
- **Collection**: `users`
- **Query**: Fetch total count of users grouped by `role` or `status`.

```typescript
// Query total active learners
const usersRef = collection(db, 'users');
const activeStudentsQuery = query(
  usersRef, 
  where('role', '==', 'student'),
  where('status', '==', 'active')
);
const snapshot = await getDocs(activeStudentsQuery);
const activeStudentCount = snapshot.size;
```

### B. Course & Topic Enrollment Summary
- **Collection**: `topics`
- **Query**: Fetch published courses to display total course catalog size and total active enrollments.

```typescript
const topicsRef = collection(db, 'topics');
const publishedTopicsQuery = query(topicsRef, where('status', '==', 'published'));
const snapshot = await getDocs(publishedTopicsQuery);
const totalCourses = snapshot.size;
```

### C. Pending Task Submissions Requiring Grading
- **Collection**: `task_submissions`
- **Query**: Fetch submissions where `status == 'submitted'` or `status == 'pending_review'`.

```typescript
const submissionsRef = collection(db, 'task_submissions');
const pendingQuery = query(
  submissionsRef,
  where('status', '==', 'submitted'),
  orderBy('submittedAt', 'desc'),
  limit(20)
);
const pendingSubmissions = (await getDocs(pendingQuery)).docs.map(doc => ({ id: doc.id, ...doc.data() }));
```

### D. Daily Engagement Metrics
- **Collection**: `daily_engagement` (Doc ID: `{dateStr}` e.g., `2026-08-31`)
- **Document Structure**:
```json
{
  "date": "2026-08-31",
  "activeUserCount": 342,
  "lessonsCompleted": 1280,
  "tasksSubmitted": 94,
  "practiceQuestionsAnswered": 4500
}
```

---

## 3. Real-Time Admin Listeners

To update the dashboard cards live without refreshing:

```typescript
import { onSnapshot, collection, query, where, limit } from 'firebase/firestore';

export function subscribeToRecentSubmissions(callback: (submissions: any[]) => void) {
  const q = query(
    collection(db, 'task_submissions'),
    where('status', '==', 'submitted'),
    orderBy('submittedAt', 'desc'),
    limit(10)
  );

  return onSnapshot(q, (snapshot) => {
    const list = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    callback(list);
  });
}
```

---

## 4. Admin CRUD Operations Checklist

| Operation | Trigger / Endpoint | Collection Modified | Description |
| :--- | :--- | :--- | :--- |
| **View System Stats** | `getDocs()` aggregation | `users`, `topics`, `tasks` | Loads overview widget counters |
| **Review Pending Tasks** | Query `status == 'submitted'` | `task_submissions` | Admin opens submission queue |
| **Trigger Recalculation** | Admin action / Cloud Function | `daily_engagement` | Recalculates daily system engagement metrics |

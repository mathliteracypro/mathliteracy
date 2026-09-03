# 02. Topics, Courses & Lessons Backend Documentation

This document specifies the backend data model, Firestore subcollection tree, and CRUD operations for managing **Topics (Courses)**, **Subtopics (Chapters)**, and **Lessons (Content)**.

---

## 1. Data Hierarchy & Firestore Tree

```
topics/{topicId}                            <-- Top-level Course document
   └── subtopics/{subtopicId}               <-- Subcollection: Chapter modules
          └── content/{contentId}           <-- Subcollection: Individual Lessons
```

---

## 2. Document Schemas

### A. Topic (Course) Document Schema (`topics/{topicId}`)

```json
{
  "id": "finance-tariffs-g11",
  "title": "Finance: Municipal Tariffs & Taxes",
  "code": "FIN-11-02",
  "description": "Comprehensive CAPS Grade 11 module covering water, electricity, and SARS tax brackets.",
  "grade": 11,
  "term": 2,
  "capsWeight": "35%",
  "category": "Finance",
  "thumbnailUrl": "https://images.unsplash.com/photo-1554224155-8d04cb21cd6c",
  "status": "published",
  "authorUid": "admin-uid-123",
  "subtopicCount": 4,
  "lessonCount": 16,
  "estimatedHours": 12,
  "tags": ["Finance", "Tariffs", "SARS", "Tax"],
  "createdAt": "2026-08-01T08:00:00.000Z",
  "updatedAt": "2026-08-31T10:00:00.000Z"
}
```

### B. Subtopic Document Schema (`topics/{topicId}/subtopics/{subtopicId}`)

```json
{
  "id": "subtopic-water-tariffs",
  "topicId": "finance-tariffs-g11",
  "title": "Block Water Tariffs & Calculation",
  "order": 1,
  "description": "Understanding sliding scale water billing in South African municipalities.",
  "lessonCount": 4,
  "createdAt": "2026-08-01T08:30:00.000Z"
}
```

### C. Content / Lesson Schema (`.../content/{contentId}`)

Supported Lesson Formats: `reading`, `video`, `quiz`, `interactive`, `qna`

```json
{
  "id": "lesson-water-calc-1",
  "topicId": "finance-tariffs-g11",
  "subtopicId": "subtopic-water-tariffs",
  "title": "Calculating Step 1 vs Step 2 Water Consumption",
  "format": "reading",
  "order": 1,
  "estimatedMinutes": 20,
  "markdownBody": "# Water Tariff Steps\nIn South Africa, municipal water uses a progressive block tariff system...",
  "videoUrl": "https://www.youtube.com/watch?v=abc123xyz",
  "quizQuestions": [
    {
      "id": "q1",
      "question": "What is the VAT rate applied to municipal services in ZAR?",
      "options": ["14%", "15%", "10%", "Exempt"],
      "correctOptionIndex": 1,
      "explanation": "Standard VAT rate in South Africa is 15%."
    }
  ],
  "status": "published",
  "createdAt": "2026-08-01T09:00:00.000Z"
}
```

---

## 3. Admin CRUD Operations

### Create Topic (Course)
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function createTopic(topicData: Omit<Topic, 'createdAt' | 'updatedAt'>) {
  const topicRef = doc(db, 'topics', topicData.id);
  const now = new Date().toISOString();
  await setDoc(topicRef, {
    ...topicData,
    createdAt: now,
    updatedAt: now
  });
}
```

### Update Topic
```typescript
import { doc, updateDoc } from 'firebase/firestore';

export async function updateTopic(topicId: string, updates: Partial<Topic>) {
  const topicRef = doc(db, 'topics', topicId);
  await updateDoc(topicRef, {
    ...updates,
    updatedAt: new Date().toISOString()
  });
}
```

### Create Lesson
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function createLesson(topicId: string, subtopicId: string, lessonData: any) {
  const lessonRef = doc(db, `topics/${topicId}/subtopics/${subtopicId}/content`, lessonData.id);
  await setDoc(lessonRef, {
    ...lessonData,
    topicId,
    subtopicId,
    createdAt: new Date().toISOString()
  });
}
```

### Delete Lesson / Subtopic / Topic
```typescript
import { doc, deleteDoc } from 'firebase/firestore';

export async function deleteTopic(topicId: string) {
  await deleteDoc(doc(db, 'topics', topicId));
}
```

---

## 4. Gemini AI Automated Lesson Generator Endpoint

Admin developers can invoke the backend Gemini endpoint to auto-generate CAPS lessons:

```typescript
const response = await fetch('/api/ai/generate-lesson', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    topic: 'Finance & Taxation',
    grade: 12,
    format: 'reading',
    difficulty: 'intermediate'
  })
});
const data = await response.json();
// Returns structured lesson JSON containing markdownBody, quizQuestions, and explanations.
```

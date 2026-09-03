# 07. Practice Arena & Study Video Catalog Backend Documentation

This document describes the question bank data model for the **AI Practice Arena** and the management of the **Study Video Catalog**.

---

## 1. Practice Arena Document Schema (`arena/{questionId}`)

```json
{
  "id": "q-arena-tariff-001",
  "topic": "Finance",
  "subtopic": "Municipal Water Tariffs",
  "grade": 11,
  "difficulty": "medium",
  "questionText": "A household consumes 24 kL of water in a month. Calculate the total cost excluding VAT based on the tariff table provided.",
  "options": [
    "R 184.50",
    "R 212.40",
    "R 244.26",
    "R 310.00"
  ],
  "correctOptionIndex": 2,
  "stepByStepSolution": "Step 1: 0 - 6 kL @ R0 = R0\nStep 2: 7 - 15 kL (9 kL @ R8.50) = R76.50\nStep 3: 16 - 24 kL (9 kL @ R18.64) = R167.76\nTotal = R76.50 + R167.76 = R244.26",
  "timesAttempted": 1420,
  "correctCount": 980,
  "tags": ["Finance", "Tariffs", "ZAR"],
  "createdAt": "2026-05-10T10:00:00.000Z"
}
```

---

## 2. Study Video Document Schema (`study_videos/{videoId}`)

```json
{
  "id": "vid-measurement-scale-map",
  "title": "Mastering Map Scales: Bar Scale vs Number Scale",
  "youtubeUrl": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
  "youtubeId": "dQw4w9WgXcQ",
  "durationSeconds": 740,
  "grade": 10,
  "topic": "Maps & Plans",
  "thumbnailUrl": "https://img.youtube.com/vi/dQw4w9WgXcQ/maxresdefault.jpg",
  "description": "Step-by-step tutorial on calculating real-world distances using CAPS bar scales.",
  "viewCount": 3890,
  "author": "MathLiteracy Pro Studio",
  "createdAt": "2026-03-01T08:00:00.000Z"
}
```

---

## 3. Admin Operations

### Add Video to Library
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function addStudyVideo(videoData: any) {
  const id = videoData.id || `vid-${Date.now()}`;
  await setDoc(doc(db, 'study_videos', id), {
    ...videoData,
    id,
    createdAt: new Date().toISOString()
  });
}
```

### Auto-Generate Practice Arena Questions via Gemini Proxy Endpoint
```typescript
const response = await fetch('/api/ai/generate-arena-questions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    topic: 'Measurement',
    subtopic: 'Packaging & Volume Calculations',
    count: 5,
    grade: 12
  })
});
const { questions } = await response.json();
// Admin can preview questions before bulk saving into 'arena' collection.
```

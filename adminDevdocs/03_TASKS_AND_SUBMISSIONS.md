# 03. Tasks & Submissions Backend Documentation

This document describes the backend architecture, schema, and workflow for managing **Tasks (Assignments, Classwork, Homework, SBA Projects)** and evaluating **Student Submissions** with manual or AI-assisted rubric grading.

---

## 1. Firestore Collections Architecture

- `tasks/{taskId}`: Task definition created by educators/admins.
- `task_questions/{questionId}`: Question bank items linked to specific tasks.
- `task_submissions/{submissionId}`: Student submission attempts, answers, uploaded media, and grading results.

---

## 2. Document Schemas

### A. Task Document (`tasks/{taskId}`)

```json
{
  "id": "task-sba-tax-2026",
  "title": "SBA Investigation: SARS Tax Brackets 2025/2026",
  "taskType": "assignment",
  "grade": 12,
  "topic": "Finance",
  "description": "Calculate tax payable for 3 different South African income brackets.",
  "markTotal": 50,
  "weightPercentage": 15,
  "startDate": "2026-08-01T00:00:00.000Z",
  "endDate": "2026-09-15T23:59:59.000Z",
  "isPublished": true,
  "allowLateSubmissions": false,
  "instructionsMarkdown": "### Instructions\n1. Use the official SARS tax table provided below...\n",
  "attachments": [
    {
      "name": "SARS_Tax_Rates_2026.pdf",
      "url": "https://storage.googleapis.com/..."
    }
  ],
  "authorUid": "admin-123",
  "createdAt": "2026-07-25T10:00:00.000Z"
}
```

### B. Task Submission Document (`task_submissions/{submissionId}`)

```json
{
  "id": "sub-user456-task-sba-tax-2026",
  "taskId": "task-sba-tax-2026",
  "studentUid": "user456",
  "studentName": "Thabo Mbeki",
  "studentEmail": "thabo@example.co.za",
  "status": "submitted",
  "submittedAt": "2026-08-30T14:22:00.000Z",
  "writtenAnswers": [
    {
      "questionNumber": 1,
      "answerText": "Taxable income is calculated after deducting pension contributions..."
    }
  ],
  "fileUrls": [
    "https://storage.googleapis.com/.../working_out.png"
  ],
  "score": 42,
  "totalPossible": 50,
  "percentage": 84,
  "feedback": "Excellent work on calculating step-down rebates!",
  "gradedBy": "ai-gemini-auto",
  "gradedAt": "2026-08-30T14:23:15.000Z",
  "rubricBreakdown": [
    { "criteria": "Tax Table Identification", "score": 10, "maxMark": 10, "comment": "Correct tier selected" },
    { "criteria": "Rebate Application", "score": 8, "maxMark": 10, "comment": "Primary rebate applied accurately" }
  ]
}
```

---

## 3. Task Management CRUD

### Create New Task Sheet
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function createAdminTask(taskData: any) {
  const taskId = taskData.id || `task-${Date.now()}`;
  const taskRef = doc(db, 'tasks', taskId);
  await setDoc(taskRef, {
    ...taskData,
    id: taskId,
    createdAt: new Date().toISOString()
  });
}
```

### Fetch All Submissions for a Task (Admin Review)
```typescript
import { collection, query, where, getDocs } from 'firebase/firestore';

export async function getTaskSubmissions(taskId: string) {
  const q = query(collection(db, 'task_submissions'), where('taskId', '==', taskId));
  const snap = await getDocs(q);
  return snap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
}
```

### Admin Manual Score / Feedback Override
```typescript
import { doc, updateDoc } from 'firebase/firestore';

export async function updateSubmissionScore(submissionId: string, score: number, feedback: string, adminUid: string) {
  const subRef = doc(db, 'task_submissions', submissionId);
  await updateDoc(subRef, {
    score,
    feedback,
    gradedBy: adminUid,
    gradedAt: new Date().toISOString(),
    status: 'graded'
  });
}
```

---

## 4. AI Automated Grading Endpoint (`server.ts`)

Admin developers can call the AI grading proxy endpoint to re-grade or batch-grade submissions:

```typescript
const response = await fetch('/api/ai/grade-task-submission', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    taskTitle: task.title,
    instructions: task.instructionsMarkdown,
    rubricMarks: task.markTotal,
    studentAnswers: submission.writtenAnswers,
    fileUrls: submission.fileUrls
  })
});
const result = await response.json();
// Returns { score, percentage, feedback, rubricBreakdown }
```

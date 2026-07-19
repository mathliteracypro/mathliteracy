# 📝 Application Page Guide: Daily Tasks & Practice

The **Daily Tasks & Practice** page houses structural learning assignments, mock examinations, and customized curriculum tasks aligned with the South African CAPS assessment schedule.

---

## 📸 Interface Preview
![Daily Tasks Workspace Layout Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Tasks+Board)
*Live UI available at: `https://mathlit.free.nf/tasks`*

---

## 💡 Page Purpose
Structured assignments form the core of school-based assessment (SBA) preparation. This interface allows students to complete scheduled school tasks, download resource attachments (PDF templates), run interactive study timers, and submit their responses for feedback.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `TaskTemplates` | `src/components/tasks/TaskTemplates.tsx` | Standard templates matching CAPS exam styles (e.g., dynamic loan tables, graph sheets). |
| `TaskTimer` | `src/components/tasks/TaskTimer.tsx` | A professional countdown timer styled to keep students focused during practice runs, supporting Pomodoro intervals. |
| `TaskCalendar` | `src/components/tasks/TaskCalendar.tsx` | Visual monthly overview plotting school deadlines, test dates, and planned practice study blocks. |
| `TaskPriority` | `src/components/tasks/TaskPriority.tsx` | Auto-sort module categorizing tasks based on due date proximity and CAPS weighting. |

---

## 🔄 Data Flow

When a user views and submits a task, the platform stores task telemetry and submission records:

```text
               [User Selects Practice Task]
                            │
               Launches Countdown Study Timer
                            │
                            ▼
               [User Uploads Completed PDF/Image]
                            │
             Saves to Firebase Storage (/tasks/)
                            │
                            ▼
         [Creates TaskSubmission in /submissions/]
                            │
         Updates task metadata: completedCount++
```

### Task Submission Schema (TypeScript Reference)
```typescript
export interface TaskSubmission {
  id: string;
  taskId: string;
  userId: string;
  userName: string;
  userEmail: string;
  submissionDate: any;        // Firestore Timestamp
  submissionText: string;     // Student working notes
  attachments: string[];      // Storage bucket URLs
  score: number;              // Graded score (-1 if ungraded)
  feedback: string;           // Teacher's feedback
  status: "pending" | "graded" | "late";
  isLate: boolean;
  createdAt: any;
}
```

---

## 🖱️ User Interactions

1. **Upload Answers**: Students can drag-and-drop images or PDFs of their completed paperwork (using the dual upload support specified in the usability guidelines).
2. **Pomodoro Tracker**: Activating the "Focused Practice" mode hides secondary sidebar widgets and starts a 25-minute practice block dedicated to solving the active problem.
3. **Template Export**: Clicking "Get Template" fetches ready-made blank spreadsheets (Excel/PDF) for calculation topics, letting students fill in values offline.

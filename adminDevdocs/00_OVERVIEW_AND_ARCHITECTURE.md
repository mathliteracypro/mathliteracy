# 00. Backend Overview & System Architecture

This document details the backend infrastructure, database configuration, security policies, and API interfaces required to construct a standalone administration dashboard for **MathLiteracy Pro**.

---

## 1. Database Infrastructure

MathLiteracy Pro uses **Firebase Firestore** as its primary document storage engine. All collections adhere to strict document structures.

### Core Firestore Collections Summary

| Collection Path | Purpose | Primary Access Mode |
| :--- | :--- | :--- |
| `users/{uid}` | User accounts, roles, profiles, and metrics | Read / Write (Admin & Owner) |
| `topics/{topicId}` | Main CAPS Curriculum Topics (Courses) | Admin Write, Public Read |
| `topics/{topicId}/subtopics/{subtopicId}` | Topic Submodules (Chapters) | Admin Write, Public Read |
| `topics/{topicId}/subtopics/{subtopicId}/content/{contentId}` | Individual Lessons (Reading, Video, Quiz, etc.) | Admin Write, Public Read |
| `tasks/{taskId}` | Assignments, Homework, SBA Projects, Classwork | Admin Write, Student Read |
| `task_submissions/{submissionId}` | Student task submissions & AI/Instructor grading | Admin Read/Write, Student Read/Write |
| `arena/{questionId}` | Practice Arena Question Bank | Admin Write, Student Read |
| `blog_posts/{postId}` | Editorial articles and announcements | Admin Write, Public Read |
| `blog_comments/{commentId}` | User comments on blog posts | Admin Moderate/Delete, User Write |
| `support_tickets/{ticketId}` | Customer & Student support requests | Admin Write/Update, User Write/Read |
| `study_videos/{videoId}` | YouTube & Video lesson catalog | Admin Write, Public Read |
| `certificates/{certificateId}` | Issued completion certificates | Admin Write/Revoke, Public Verify |
| `notifications/{notificationId}` | System & user alert notifications | Admin Write (Broadcast), User Read |
| `sent_emails/{emailId}` | System audit log of sent email dispatches | Admin Write/Read |
| `onboarding/{uid}` | User onboarding wizard state tracking | Admin Read, User Write |

---

## 2. Authentication & Role-Based Access Control (RBAC)

### User Roles Hierarchy

1. **`admin`**: Full system read/write access. Can manage users, courses, tasks, support tickets, emails, and system configs.
2. **`instructor`**: Educator role. Can create/manage tasks, grade submissions, publish lessons, and view assigned topic statistics.
3. **`moderator`**: Content moderator. Can approve/delete blog comments and respond to support tickets.
4. **`student`**: Learner role. Enrolls in topics, completes lessons, submits tasks, practices in arena.
5. **`parent`**: Guardian role. Associated with student accounts to view progress reports.

### Admin Verification Rules

Admin privilege check follows two primary patterns:
1. **Firestore `role` Field**: `doc.data().role === 'admin'`
2. **Environment Allowlist**: `VITE_ADMIN_EMAILS` environment variable (comma-separated email list).

```typescript
// Example Admin Check Function
export function isAdminUser(email?: string | null, role?: string | null): boolean {
  if (role === 'admin') return true;
  const adminEmails = (import.meta.env.VITE_ADMIN_EMAILS || '').split(',').map(e => e.trim().toLowerCase());
  return Boolean(email && adminEmails.includes(email.toLowerCase()));
}
```

---

## 3. Backend Proxy Endpoints (`server.ts`)

The Express backend handles AI model proxying and secure SMTP dispatches.

| Method | Route | Description | Input Payload |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/ai/generate-lesson` | Generates CAPS-aligned lesson content via Gemini | `{ topic, grade, format, difficulty }` |
| `POST` | `/api/ai/generate-arena-questions` | Generates practice question sets | `{ topic, subtopic, count, grade }` |
| `POST` | `/api/ai/generate-task` | Creates SBA assignments with rubrics | `{ topic, grade, taskType, markTotal }` |
| `POST` | `/api/ai/grade-task-submission` | Evaluates student answers against rubric | `{ task, submissionText, fileUrls }` |
| `POST` | `/api/mail/send` | Sends transactional email via SMTP Nodemailer | `{ to, subject, templateId, variables }` |

---

## 4. Common Document Timestamps Standard

All documents created or updated in Firestore MUST maintain standardized ISO timestamp strings:

```json
{
  "createdAt": "2026-08-31T11:20:00.000Z",
  "updatedAt": "2026-08-31T11:30:00.000Z"
}
```

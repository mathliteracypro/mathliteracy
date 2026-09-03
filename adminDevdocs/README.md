# MathLiteracy Pro - Admin Developer Backend Documentation

> [!CAUTION]
> **CONFIDENTIAL & PROPRIETARY PROPERTY OF MATHLITERACY PRO**  
> **RESTRICTED ACCESS**: The documentation and architectural specifications contained within this directory (`/documentary/adminDevdocs/`) are strictly reserved for authorized platform administrators and verified lead developers. **Access, distribution, copying, or utilization by unidentified, unauthenticated, or unauthorized users is strictly forbidden.**

Welcome to the backend architecture and CRUD documentation for **MathLiteracy Pro**. This directory provides standalone admin developers with a complete, independent reference guide for integrating with the Firestore database, backend services, and API endpoints required to build a standalone administration dashboard.

---

## 📚 Documentation Index

| File | Page / Feature Area | Key Operations & Collections |
| :--- | :--- | :--- |
| [`00_OVERVIEW_AND_ARCHITECTURE.md`](./00_OVERVIEW_AND_ARCHITECTURE.md) | **Architecture & Security** | Firestore collections schema, Auth & RBAC rules, South African CAPS metadata, server API specs |
| [`01_DASHBOARD_ANALYTICS.md`](./01_DASHBOARD_ANALYTICS.md) | **Dashboard Page** | Aggregated stats, active task counters, daily engagement tracking (`users`, `tasks`, `daily_engagement`) |
| [`02_TOPICS_COURSES_LESSONS.md`](./02_TOPICS_COURSES_LESSONS.md) | **Topics & Lessons Pages** | CRUD for CAPS Topics, Subtopics, and multi-format Content (`topics`, `subtopics`, `content`) |
| [`03_TASKS_AND_SUBMISSIONS.md`](./03_TASKS_AND_SUBMISSIONS.md) | **Tasks & Submissions Pages** | Task sheet CRUD, student submissions, AI rubric auto-grading (`tasks`, `task_submissions`, `task_questions`) |
| [`04_USER_AND_ROLE_MANAGEMENT.md`](./04_USER_AND_ROLE_MANAGEMENT.md) | **Users & Profiles Pages** | Student/Parent/Teacher profiles, role assignment, status toggling (active/banned/suspended) (`users`) |
| [`05_INSTRUCTORS_MANAGEMENT.md`](./05_INSTRUCTORS_MANAGEMENT.md) | **Instructors Pages** | Instructor profile management, topic assignments, rating metrics (`users`, `instructor_profiles`) |
| [`06_BLOG_AND_NEWSLETTER.md`](./06_BLOG_AND_NEWSLETTER.md) | **Blog & Newsletter Pages** | Post creation, comment moderation, newsletter subscriber lists (`blog_posts`, `blog_comments`, `newsletters`) |
| [`07_PRACTICE_ARENA_AND_STUDY.md`](./07_PRACTICE_ARENA_AND_STUDY.md) | **Arena & Study Pages** | Practice question bank CRUD, study video metadata management (`arena`, `study_videos`) |
| [`08_CERTIFICATES_MANAGEMENT.md`](./08_CERTIFICATES_MANAGEMENT.md) | **Certificates Page** | Certificate issuance, verification hashing, revocation, topic completion criteria (`certificates`) |
| [`09_SUPPORT_AND_TICKETS.md`](./09_SUPPORT_AND_TICKETS.md) | **Support & Help Pages** | Support ticket inbox, ticket status updates, contact settings (`support_tickets`, `contact_settings`) |
| [`10_NOTIFICATIONS_AND_MAILER.md`](./10_NOTIFICATIONS_AND_MAILER.md) | **Notifications & Mailer** | System notifications dispatch, broadcast management, sent logs (`notifications`, `sent_emails`) |
| [`11_AUTHENTICATION_AND_ONBOARDING.md`](./11_AUTHENTICATION_AND_ONBOARDING.md) | **Auth & Onboarding Pages** | Google OAuth & Student ID auth flows, onboarding step state management (`onboarding`) |

---

## 🚀 Quick Tech Stack Reference for Admin Developers

- **Database**: Firebase Firestore (`ai-studio-533c50d5-289e-4a4c-9b15-34528af3bbd4`)
- **Authentication**: Firebase Authentication (Google OAuth & Student ID auth provider fallback)
- **Backend Proxy Server**: Express.js (`/api/ai/*`, `/api/mail/*`)
- **Curriculum Alignment**: South African CAPS (Grade 10, 11, 12 Mathematical Literacy)

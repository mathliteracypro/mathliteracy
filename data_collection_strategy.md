# MathLiteracy Pro - Course & Lesson Data Collection Strategy

## Overview

MathLiteracy Pro employs a hierarchical, offline-capable, real-time data collection strategy powered by Firebase Firestore. Data models are designed in strict alignment with the South African CAPS Mathematical Literacy curriculum standards.

---

## 📁 1. Hierarchical Firestore Collection Structure

```
topics/{topicId}
  ├── subtopics/{subtopicId}
  │     └── content/{contentId}
  └── enrolled_students/{userId}

course_progress/{courseId}_{userId}
users_progress/{userId}
```

### Collection Path Specifications

| Collection / Subcollection Path | Purpose | Key Fields |
| :--- | :--- | :--- |
| `topics/{topicId}` | Primary Course Containers | `topic_name`, `grade_level`, `category`, `short_description`, `thumbnail`, `is_published`, `total_enrollments`, `subtopic_count`, `created_at` |
| `topics/{topicId}/subtopics/{subtopicId}` | Chapters / Modules | `subtopic_title`, `subtopic_description`, `subtopic_order`, `is_published`, `content_count` |
| `topics/{topicId}/subtopics/{subtopicId}/content/{contentId}` | Lesson Units (Video, Reading, Quiz) | `content_title`, `content_type`, `content` (Markdown), `duration_minutes`, `video_url`, `quiz`, `content_order` |
| `topics/{topicId}/enrolled_students/{userId}` | Course Enrollments | `user_id`, `enrollment_status`, `progress`, `progress_percentage`, `enrollment_date`, `last_accessed` |
| `course_progress/{courseId}_{userId}` | Course Level Progress Tracking | `courseId`, `userId`, `completedLessons`, `totalLessons`, `progressPercentage`, `lastAccessed` |
| `users_progress/{userId}` | Global Learner Profile Progress | `completed_content`, `total_lessons`, `total_study_time`, `average_quiz_score`, `overall_percentage` |

---

## ⚡ 2. Index Requirements & Query Optimization

To prevent unindexed query errors, the following composite indexes are configured in Firestore:

1. **`topics`**:
   - `is_published` ASC, `grade_level` ASC, `created_at` DESC
2. **`topics/{topicId}/subtopics`**:
   - `is_published` ASC, `subtopic_order` ASC
3. **`topics/{topicId}/subtopics/{subtopicId}/content`**:
   - `is_published` ASC, `content_order` ASC
4. **`topics/{topicId}/enrolled_students`** (Collection Group Query):
   - `user_id` ASC, `enrolledAt` DESC

---

## 🚀 3. Parallel Data Loading & Real-Time Sync Strategy

### Parallel Loader Pattern (`loadCourseDetail`)

Data is fetched concurrently using `Promise.all` to minimize time-to-first-render:

```typescript
const [course, subtopics, enrollmentsCount, progress] = await Promise.all([
  fetchCourse(courseId),
  fetchSubtopics(courseId),
  getDocs(collection(db, 'topics', courseId, 'enrolled_students')).then(s => s.size),
  userId ? fetchCourseProgress(courseId, userId) : Promise.resolve(null)
]);
```

### Real-Time Synchronization

Active listeners (`onSnapshot`) track live changes to enrollment stats, progress updates, and course content without requiring page reloads:

```typescript
// Course document subscriber
subscribeToCourse(courseId, (data) => updateCourseState(data));

// Enrollment count subscriber
subscribeToEnrollments(courseId, (count) => updateEnrollmentCount(count));

// Learner progress subscriber
subscribeToProgress(courseId, userId, (progressData) => updateProgress(progressData));
```

---

## 🛡️ 4. Security & Access Control

- **Master Security Rules**: Configured in `firestore.rules` using Attribute-Based Access Control (ABAC).
- **Collection Group Rule**: `match /{path=**}/enrolled_students/{enrollmentId}` enforces secure read/write verification for learner enrollments.
- **Data Isolation**: User progress documents (`users_progress/{userId}`) restrict modification to authenticated owners and system administrators.

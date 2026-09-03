# 📚 MathLiteracy Pro: Courses, Course Details & Lessons Documentation

This documentation provides a comprehensive guide to the data collection strategy, schema structures, queries, and state management lifecycle for **Courses (Topics)**, **Course Details (Subtopics)**, and **Lessons (Content Units)** in MathLiteracy Pro.

---

## 📐 1. Architecture & Collection Hierarchy

MathLiteracy Pro uses a nested Firestore hierarchy combined with top-level query views for efficient pagination, real-time subscription, and transaction-based progress tracking.

```
Firestore Collection Topology:
├── topics/{topicId}                                     [Course Containers]
│   ├── subtopics/{subtopicId}                           [Chapters / Modules]
│   │   └── content/{contentId}                          [Lesson Units (Video/Reading/Quiz)]
│   └── enrolled_students/{enrollmentId}                 [Topic-Level Enrollments]
├── enrollments/{userId}_{topicId}                       [Top-Level Enrollments Index]
├── course_progress/{userId}_{courseId}                  [Course Progress Metrics]
└── users_progress/{userId}                             [Global Learner Progress Profile]
```

---

## 🎓 2. Courses (`topics` Collection)

### 2.1 Firestore Schema: `topics/{topicId}`

Each course in MathLiteracy Pro is represented as a **Topic** document in the top-level `topics` collection.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | Unique Topic document identifier |
| `topic_name` / `title` | `string` | Display name of the course (e.g. *"Financial Documents & Tax Rates"*) |
| `course_code` / `topic_code` | `string` | CAPS alignment code (e.g. *"FIN-G11-01"*) |
| `grade_level` | `string` | Target grade: `"Grade 10"`, `"Grade 11"`, `"Grade 12"` |
| `category` | `string` | CAPS Subject Area: `"Finance"`, `"Measurement"`, `"Mapwork"`, `"Data Handling"` |
| `difficulty` / `difficulty_level` | `string` | Difficulty tier: `"Beginner"`, `"Intermediate"`, `"Advanced"` |
| `short_description` | `string` | Brief course synopsis for card displays |
| `description` | `string` | Full Markdown syllabus overview and learning objectives |
| `thumbnail` / `image_url` | `string` | Cover image asset URL |
| `is_published` | `boolean` | Publication state flag (`true` to render in catalog) |
| `total_enrollments` | `number` | Counter incremented via Firestore transactions |
| `subtopic_count` | `number` | Count of subtopics/chapters within this course |
| `created_by` | `string` | Admin/Instructor User ID |
| `created_at` | `string` (ISO) | ISO timestamp of topic creation |
| `updated_at` | `string` (ISO) | ISO timestamp of last topic modification |

### 2.2 Course Data Collection Methods

In `CourseService` (`src/services/courseService.ts`):

- **`getCoursesWithProgress(userId, options)`**:
  - Queries `topics` filtered by `is_published == true`, optional `grade_level`, and `category`.
  - Supports pagination using `limitQuery()` and `startAfter(lastDoc)`.
  - Merges each topic document with the learner's progress metrics (`isEnrolled`, `completedLessons`, `totalLessons`, `progressPercentage`).
- **`enrollInCourse(userId, courseId)`**:
  - Atomic Firestore transaction that creates the enrollment record in `enrollments/{userId}_{courseId}`, initializes `users_progress/{userId}`, increments `total_enrollments` on the topic document, and updates the user's course stats.

---

## 📖 3. Course Details (`topics/{topicId}/subtopics`)

### 3.1 Subtopic Schema: `topics/{topicId}/subtopics/{subtopicId}`

Subtopics represent structural modules or chapters within a course.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | Subtopic document ID |
| `subtopic_title` / `title` | `string` | Subtopic/Chapter title (e.g. *"Payslips & Unemployment Insurance Fund (UIF)"*) |
| `subtopic_description` | `string` | Chapter description and concept summary |
| `subtopic_order` | `number` | Numerical display order within the course |
| `is_published` | `boolean` | Publication status flag |
| `content_count` | `number` | Total number of lesson content units in this subtopic |

### 3.2 Parallel Data Loading (`TopicService`)

In `TopicService` (`src/services/topicService.ts`), loading course details utilizes `Promise.all` for fast time-to-first-render:

```typescript
// 1. Concurrent fetch of Topic header, Subtopics list, and Enrollment status
const [topic, subtopics, enrollment] = await Promise.all([
  this.fetchTopic(topicId),
  this.fetchSubtopics(topicId),
  this.fetchEnrollmentStatus(topicId, userId),
]);

// 2. Concurrent fetch of all content items across all subtopics
const contentPromises = subtopics.map((sub) => this.fetchContent(topicId, sub.id));
const subtopicContents = await Promise.all(contentPromises);
const content = subtopicContents.flat();
```

### 3.3 Real-Time Subscription Lifecycle

`topicService.subscribeToTopic(topicId, userId, onData, onError)` establishes concurrent listeners:
1. `onSnapshot` on `topics/{topicId}` for live header changes.
2. `onSnapshot` on `topics/{topicId}/subtopics` (ordered by `subtopic_order asc`).
3. Dynamic subcollection `onSnapshot` on each subtopic's `content` collection.
4. `onSnapshot` on `topics/{topicId}/enrolled_students/enr_{userId}_{topicId}` for learner progress.

---

## 🎥 4. Lessons (`topics/{topicId}/subtopics/{subtopicId}/content`)

### 4.1 Lesson Schema: `content/{contentId}`

Content documents hold individual interactive or video learning items.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | Content unit ID |
| `content_title` / `title` | `string` | Lesson title (e.g. *"Calculating Gross vs. Net Pay"*) |
| `content_type` | `string` | Lesson mode: `"video"`, `"reading"`, `"quiz"`, `"interactive"`, `"qa"` |
| `content` / `body` / `markdown` | `string` | Full Markdown instruction body with KaTeX math formula support |
| `duration_minutes` | `number` | Estimated duration in minutes (e.g. `15`) |
| `video_url` | `string` | YouTube URL (`youtube.com/watch?v=` or `youtu.be/`) |
| `content_order` | `number` | Sequence order within the subtopic |
| `is_published` | `boolean` | Visibility toggle |
| `quiz` | `object` | Optional quiz data structure containing `questions`, `passing_score`, and `time_limit` |

### 4.2 Lesson Progress Data Collection & Updating

Lesson progress is tracked across two primary locations in Firestore:
1. `course_progress/{userId}_{courseId}`: Detail document storing per-lesson status map (`lessons.${lessonId}`), completion flags, `timeSpent`, and last positions.
2. `users_progress/{userId}`: Learner-wide array of completed lesson IDs (`completed_content`) and quiz scores.

#### Lesson Completion Transaction (`CourseService.completeLesson`)
```typescript
await runTransaction(db, async (transaction) => {
  // 1. Fetch current users_progress document
  // 2. Append lessonId to completed_content array
  // 3. Write timestamped content_progress entry
  // 4. Update course_progress document
  // 5. Increment user study_stats (lessonsCompleted & lastStudyDate)
});
```

---

## 📊 5. Aggregated Course & Duration Stats

To display aggregated statistics across enrolled courses on the Dashboard, `dashboardStatsService` (`src/services/dashboardStatsService.ts`) calculates:

$$\text{Total Duration (Hours)} = \frac{\sum \text{duration\_minutes of all published content across enrolled topics}}{60}$$

$$\text{Overall Progress (\%)} = \frac{\sum \text{progress\_percentage of enrolled topics}}{\text{Count of enrolled topics}}$$

### Local Caching Strategy
- `getTopicDurationsWithCache(topicIds)` stores topic duration calculations in `localStorage` under `topic_durations_cache` with a **5-minute TTL** to minimize unnecessary subcollection reads.

---

## 🛡️ 6. Security & Rule Validation

Access is governed by `firestore.rules`:
- **Read Access**: Anyone can read published topics, subtopics, and content.
- **Write Access**: Restricted strictly to admin and instructor roles.
- **Learner Progress**: Users can only create and write to their own enrollment (`enrollments/{userId}_{topicId}`) and progress (`users_progress/{userId}`, `course_progress/{userId}_{courseId}`) records.

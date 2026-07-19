# 🗄️ Firebase Firestore Database Schema Document

The MathLiteracy Pro database uses a robust NoSQL structure designed to handle complex course content catalogs, user tracking subcollections, and task submissions. This schema ensures highly queryable data structures, strong normalization of metadata, and reliable subcollection paths.

---

## 🗺️ Collections Map

```text
/users/ (uid)
  ├── /video_progress/ (videoId)
  └── /streak_history/ (dateId)

/topics/ (topicId)
  ├── /subtopics/ (subtopicId)
  │     └── /content/ (contentId)
  └── /enrolled_students/ (uid)

/instructors/ (instructorId)

/blog_posts/ (postId)
  └── /comments/ (commentId)

/tasks/ (taskId)
  ├── /submissions/ (submissionId)
  └── /questions/ (questionId)

/system_notifications/ (notificationId)

/certificates/ (certificateId)
```

---

## 📑 Collection-Level Details

### 1. `users` Collection
- **Path**: `/users/{uid}`
- **Purpose**: Stores student credentials, badges earned, totals, and setup profiles.

| Field Name | Data Type | Description |
| :--- | :--- | :--- |
| `username` | `string` | Unique profile handle |
| `email` | `string` | Registered contact e-mail address |
| `first_name` | `string` | First name of student |
| `last_name` | `string` | Surname of student |
| `account_type` | `string` | User role restriction: `"free" \| "admin"` |
| `status` | `string` | Account standing: `"active" \| "suspended" \| "banned"` |
| `current_streak`| `number` | Consecutive daily study sessions active |
| `certificates` | `array<string>` | IDs of earned certificates |
| `badges` | `array<string>` | IDs of unlocked game badges |
| `totalStudyTime`| `number` | Total overall course consumption in minutes |

#### 📂 Subcollection: `video_progress`
- **Path**: `/users/{uid}/video_progress/{videoId}`
- **Purpose**: Track position timestamps for lesson videos.
- **Fields**:
  - `secondsWatched`: `number` (current playback second marker)
  - `durationSeconds`: `number` (total duration)
  - `isCompleted`: `boolean`
  - `lastAccessed`: `timestamp`

---

### 2. `topics` Collection
- **Path**: `/topics/{topicId}`
- **Purpose**: High-level courses mapping major CAPS themes (e.g. Finance, Mapwork).

| Field Name | Data Type | Description |
| :--- | :--- | :--- |
| `course_code` | `string` | Short reference string: e.g. "CAPS-FIN12" |
| `course_name` | `string` | Full academic name of topic |
| `grade_level` | `string` | Associated school grade: `"Grade 10" \| "Grade 11" \| "Grade 12"` |
| `caps_weighting`| `number` | Official CAPS weighting percentage |
| `duration_hours`| `number` | Estimated duration to complete syllabus |
| `status` | `string` | Visibility: `"draft" \| "active" \| "archived"` |

#### 📂 Subcollection: `subtopics`
- **Path**: `/topics/{topicId}/subtopics/{subtopicId}`
- **Purpose**: Chapters breaking down the main topic.
- **Fields**:
  - `module_title`: `string`
  - `module_description`: `string`
  - `module_order`: `number`

#### 📂 Subcollection: `enrolled_students`
- **Path**: `/topics/{topicId}/enrolled_students/{uid}`
- **Purpose**: Records specific student enrollments. Priority path for enrollment reads.
- **Fields**:
  - `user_id`: `string`
  - `enrollment_date`: `timestamp`
  - `progress_percentage`: `number`

---

### 3. `tasks` Collection
- **Path**: `/tasks/{taskId}`
- **Purpose**: Homework templates, deadlines, and grades.

| Field Name | Data Type | Description |
| :--- | :--- | :--- |
| `title` | `string` | Name of task |
| `topic` | `string` | Related CAPS syllabus topic |
| `dueDate` | `timestamp`| Official cutoff date for submissions |
| `requiresSubmission`| `boolean` | Requires PDF/image file upload |
| `points` | `number` | Maximum score of assignment |

#### 📂 Subcollection: `submissions`
- **Path**: `/tasks/{taskId}/submissions/{submissionId}`
- **Purpose**: Student-submitted homework files and grades.
- **Fields**:
  - `userId`: `string`
  - `submissionText`: `string`
  - `attachments`: `array<string>` (URLs from Storage buckets)
  - `score`: `number` (grade result out of points)
  - `status`: `string` (`"pending" \| "graded"`)

---

## 🧬 Relations & Foreign Key Map

```text
User Profile (ID)  <══ (1:N) ══>  Enrollments (user_id)
Course Topic (ID)  <══ (1:N) ══>  Subtopics (course_id)
Subtopic (ID)      <══ (1:N) ══>  Lessons / Content (module_id)
Task Assignment (ID)<══ (1:N) ══>  TaskSubmissions (taskId)
```

# 📖 Application Page Guide: Courses Catalog

The **Courses** page displays a structured, interactive list of South African CAPS-aligned Mathematical Literacy topics. It provides a progressive breakdown from high-level topics (e.g., Financial Mathematics, Maps and Scales) down to modular subtopics and individual text or video lessons.

---

## 📸 Interface Preview
![Courses Catalog Layout Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Courses+Layout)
*Live UI available at: `https://mathlit.free.nf/courses`*

---

## 💡 Page Purpose
The catalog helps students find exact CAPS topics, view weightings (e.g., CAPS Weighting: 35%), filter content by Grade level (Grades 10, 11, or 12), and enroll with a single click. It serves as the primary route to start and track learning goals.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `CourseCard` | `src/components/course/CourseCard.tsx` | Reusable presentation component displaying the thumbnail, CAPS weighting, difficulty level, and user enrollment status. |
| `CourseDetails` | `src/pages/CourseDetails.tsx` | Expanded view of a chosen course/topic, presenting full chapters, modules, duration breakdowns, reviews, and interactive syllabus list. |
| `TopicDetail` | `src/pages/TopicDetail.tsx` | Dynamic layout to display the chapters, modules, and progress items specific to one major CAPS topic. |
| `SubtopicDetail` | `src/pages/SubtopicDetail.tsx` | High-fidelity screen displaying detailed concepts, downloadable guides, reading text, and formulas for a specific subtopic. |

---

## 🔄 Data Flow

The courses interface connects with Firestore `topics` to list active curriculum nodes.

```text
       [Firestore /topics Collection]
                     │
         Query Filters applied in UI:
         ├── Grade Level (10 | 11 | 12)
         ├── Difficulty (Beginner | Intermediate | Advanced)
         └── Search Query (String regex match)
                     │
                     ▼
      [Component: CourseCard Render Engine]
                     │
       User click "Enroll" or "Resume"
                     │
                     ▼
  Writes to /topics/{topicId}/enrolled_students/{uid}
```

### Course / Topic Type Schema (TypeScript Reference)
```typescript
export interface Course {
  id: string;
  course_code: string;
  course_name: string;
  course_slug: string;
  description: string;
  short_description: string;
  thumbnail: string;
  image_url: string;
  video_url: string; // Course intro video
  category: string;
  grade_level: "Grade 10" | "Grade 11" | "Grade 12";
  difficulty_level: "beginner" | "intermediate" | "advanced";
  caps_weighting: number; // CAPS weighting percentage e.g. 25
  duration_hours: number;
  duration_minutes: number;
  module_count: number;
  views: number;
  average_rating: number;
  total_enrollments: number;
  is_featured: boolean;
  is_published: boolean;
  status: "draft" | "active" | "archived";
  primary_instructor_id: string;
  primary_instructor_name: string;
  created_at: string;
  published_at?: string;
  updated_at: string;
}
```

---

## 🖱️ User Interactions

1. **Category & Grade Filtering**: Multi-select pill controls let users instantly refine courses based on Grade level (e.g., CAPS Grade 12 Paper 1 concepts) or category (e.g. Finance, Mapwork).
2. **Instant Enrollment**: Enrolling creates a student-specific tracking sub-document. The "Enroll" button triggers a smooth CSS transition and immediately converts to "Resume Course".
3. **Curriculum Accordion**: Inside `CourseDetails`, subtopics can be expanded to reveal the lessons, video links, readings, or quizzes contained within.

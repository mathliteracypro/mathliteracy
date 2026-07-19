# 🎓 Application Page Guide: Instructors Hub

The **Instructors Hub** hosts professional credentials and profiles of active educators on the MathLiteracy Pro platform. It allows students to research the authors of their courses, view teacher performance reviews, and explore additional modules authored by their favorite instructors.

---

## 📸 Interface Preview
![Instructors Workspace Layout Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Instructors+Hub)
*Live UI available at: `https://mathlit.free.nf/instructors`*

---

## 💡 Page Purpose
Building trust is essential in online learning. The Instructors Hub establishes teacher credibility by showcasing academic credentials, certified experience (e.g., "15 Years High School Experience"), student enrollment statistics, and aggregate ratings.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `InstructorCard` | `src/components/instructors/InstructorCard.tsx` | Summary display card showing the profile picture, core specialization (e.g., Financial Literacy, Mapwork Specialist), and action link. |
| `InstructorStats` | `src/components/instructors/InstructorStats.tsx` | Data-heavy dashboard widget showcasing key statistics: courses published, active students enrolled, and review averages. |
| `FeaturedInstructors` | `src/components/instructors/FeaturedInstructors.tsx` | Homepage highlight carousel featuring top-rated educators. |
| `BecomeInstructor` | `src/components/instructors/BecomeInstructor.tsx` | Automated portal and application form for qualified South African educators to apply for teaching clearance. |

---

## 🔄 Data Flow

Educators are stored in a root level `/instructors` collection. Their active course catalog is queried dynamically.

```text
               [User Navigates to Instructor Profile]
                                  │
                  Fetches Instructor Document:
                   /instructors/{instructorId}
                                  │
                                  ▼
                Queries all courses matching:
           /topics where primary_instructor_id == ID
                                  │
                                  ▼
                   Combines ratings & students:
                   - Calculates total reach
                   - Lists authored courses
```

### Instructor Schema (TypeScript Reference)
```typescript
export interface Instructor {
  id: string;
  first_name: string;
  last_name: string;
  email: string;
  specialization: string;     // e.g. "CAPS Finance, Measurement"
  bio: string;
  profile_image: string;
  experience_years: number;
  is_featured: boolean;
  status: "active" | "inactive" | "pending";
  social_linkedin?: string;
  youtube_channel_url?: string;
  created_at: string;
  updated_at: string;
}
```

---

## 🖱️ User Interactions

1. **Teacher Rating Breakdown**: Students can view complete rating reviews. Clicking on stars displays a bar graph of 1-star through 5-star student review counts.
2. **Apply to Teach**: Certified teachers can click "Apply as Instructor", which opens a multipage application form requesting school details, SACE registration numbers, and teaching samples.
3. **Course Quick-Links**: Clicking on any authored course within an instructor's profile smoothly routes the student directly to `/courses/{courseId}` with full context pre-loaded.

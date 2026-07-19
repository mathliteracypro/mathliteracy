# 👤 Application Page Guide: Profile & Settings

The **Profile & Settings** view acts as a personal transcript page for the student. It manages account details, tracks achievement milestones (Badges), compiles earned CAPS completion certificates, and controls user preferences.

---

## 📸 Interface Preview
![Profile View Layout Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Profile+Hub)
*Live UI available at: `https://mathlit.free.nf/profile`*

---

## 💡 Page Purpose
The Profile page serves as verification of earned academic progress. It allows students to visually inspect their completed courses, review historical test grades, export PDF certificates with valid cryptographic verification hashes, and tweak UI preferences.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `ProfileHeader` | `src/components/profile/ProfileHeader.tsx` | Main display showing the user avatar, full name, country/province, onboarding date, and account status tags. |
| `ProfileOverviewTab` | `src/components/profile/ProfileOverviewTab.tsx` | High-level bento grid summarizing active streaks, lesson completions, and study durations. |
| `ProfileProgressTab` | `src/components/profile/ProfileProgressTab.tsx` | Advanced study charts rendered using Recharts/D3 showing historical day-by-day study minutes. |
| `ProfileAchievementsTab` | `src/components/profile/ProfileAchievementsTab.tsx` | Gamified milestones display listing unlocked and locked badge rewards (e.g. "Finance Champ", "Weekly Streak Master"). |

---

## 🔄 Data Flow

When a user modifies their settings or updates their avatar, the client fires atomic writes to the user's document.

```text
               [User Triggers Settings Update]
                             │
            Validates changes with local helpers
                             │
                             ▼
         [Writes to Firestore: /users/{userId}]
                             │
        ┌────────────────────┴────────────────────┐
        ▼                                         ▼
   Updates Local Auth Context               Emits "sync_success" Toast
   (Immediate UI Redraw)                    (Feedback banner)
```

### Profile Data Structure (TypeScript Reference)
```typescript
export interface UserProfile {
  id: string;
  username: string;
  email: string;
  first_name: string;
  last_name: string;
  phone: string;
  profile_image: string;
  bio: string;
  country: string;
  city: string;
  account_type: "free" | "admin";
  status: "active" | "inactive" | "suspended" | "pending" | "banned";
  is_admin: boolean;
  school?: string;
  grade?: string; // Grade 10, Grade 11, Grade 12
  province?: string;
  preferred_difficulty?: "beginner" | "intermediate" | "advanced";
  totalCoursesEnrolled?: number;
  totalLessonsCompleted?: number;
  totalStudyTime?: number; // total minutes
  certificates?: string[]; // IDs of earned certificates
}
```

---

## 🖱️ User Interactions

1. **Certificate Verification Export**: Earned certificates feature a distinct "Verify Link". Clicking it generates a copyable public URL (linked to `https://mathlit.free.nf/verify/{certificateHash}`) that can be sent to parents or educators to confirm its authenticity.
2. **Preference Adjustments**: Users can toggle platform-wide UI preferences (e.g., opting out of advertisements, subscribing to newsletter circulars, choosing their preferred difficulty tier).
3. **Avatar Upload**: Supports quick-swapping profile pictures using our responsive canvas image selectors.

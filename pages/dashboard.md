# 🖥️ Application Page Guide: Dashboard

The **Dashboard** serves as the central hub of the MathLiteracy Pro application. Upon logging in, users are presented with a unified portal showcasing their active study stats, current streaks, recent video logs, and upcoming learning tasks. It acts as an orchestrator of user progression.

---

## 📸 Interface Preview
![Dashboard Wireframe Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Dashboard+Layout)
*Live UI available at: `https://mathlit.free.nf/dashboard`*

---

## 💡 Page Purpose
The primary objective of the Dashboard is to foster learning consistency (streaks) and provide micro-feedback on CAPS performance targets. By combining dynamic data visualization with actionable insights, the user is never left wondering "what should I study next?"

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `DashboardWidgets` | `src/components/dashboard/DashboardWidgets.tsx` | Main grid of user telemetry: total study minutes, lessons completed, certifications earned, and global streak counter. |
| `EmptyState` | `src/components/dashboard/EmptyState.tsx` | Informative state displayed when the user is not enrolled in any courses, presenting onboarding directions. |
| `WidgetLoader` | `src/components/dashboard/WidgetLoader.tsx` | Smooth skeletons matching the layout for zero cumulative layout shift (CLS) during data loads. |
| `DashboardErrorBoundary` | `src/components/dashboard/DashboardErrorBoundary.tsx` | Localized error boundaries to prevent full-screen page crashes on database connection glitches. |

---

## 🔄 Data Flow

The Dashboard queries multiple collections simultaneously to formulate its comprehensive stats panel.

```text
                  [Firebase Auth (CurrentUser)]
                               │
               ┌───────────────┼───────────────┐
               ▼               ▼               ▼
        /users/{uid}     /enrollments    /video_history
               │               │               │
               ▼               ▼               ▼
          UserProfile     Active Courses   Recent Progress
               │               │               │
               └───────────────┼───────────────┘
                               ▼
                    [Dashboard State Merging]
                               │
                               ▼
                 Recharts & Stat Card Renderers
```

### Dynamic Code Reference (Dashboard Data Fetching)
```typescript
// Sample implementation demonstrating dynamic data fetching
export const useDashboardStats = (userId: string) => {
  const [stats, setStats] = useState<DashboardStats | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!userId) return;

    const fetchStats = async () => {
      try {
        const userDoc = await getDoc(doc(db, "users", userId));
        const enrollmentsQuery = query(
          collection(db, "enrollments"),
          where("user_id", "==", userId)
        );
        const enrollSnap = await getDocs(enrollmentsQuery);
        
        // Compute total active lessons and study times
        const courses = enrollSnap.docs.map(d => d.data());
        const totalMinutes = courses.reduce((sum, c) => sum + (c.progress_minutes || 0), 0);
        
        setStats({
          streak: userDoc.data()?.current_streak || 0,
          minutesStudied: totalMinutes,
          courseCount: courses.length,
          completedCount: courses.filter(c => c.status === "completed").length
        });
      } catch (err) {
        console.error("Failed to aggregate dashboard telemetry", err);
      } finally {
        setLoading(false);
      }
    };

    fetchStats();
  }, [userId]);

  return { stats, loading };
};
```

---

## 🖱️ User Interactions

1. **Streak Validation**: Users hover over the streak widget to view their streak calendar (e.g., historical attendance from `users/{uid}/streak_history`).
2. **Recent Video Resume**: A specialized widget showcases the last video the user was actively watching (fetched from local IndexedDB/Firestore `video_history` subcollection). Clicking "Resume" redirects the user directly to the `/study` player at the exact timestamp saved.
3. **Task Quick-Actions**: Uncompleted high-priority daily study tasks are listed. Clicking a task opens the `/tasks` interactive interface with active task templates loaded.

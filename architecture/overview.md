# 🧱 High-Level Systems Architecture Overview

This document presents a comprehensive, low-level look at the component hierarchies, state contexts, and execution lifecycles that power the MathLiteracy Pro application.

---

## 🗺️ System Component Tree Architecture

The frontend is constructed using a decoupled, context-driven component architecture. Layout elements are separated from domain-specific features:

```text
                                [src/main.tsx]
                                      │
                                 [src/App.tsx]
                                      │
                         ┌────────────┴────────────┐
                         ▼                         ▼
                 [AppContextProvider]     [EditorContextProvider]
                         │                         │
            ┌────────────┼─────────────────────────┼────────────┐
            ▼            ▼                         ▼            ▼
        [Sidebar]    [Header]               [AppRoutes / Router] [Footer]
                                                   │
                ┌───────────────┬──────────────────┼───────────────┬──────────────┐
                ▼               ▼                  ▼               ▼              ▼
           /dashboard        /courses            /study          /tasks        /profile
          (Dashboard)   (Courses/Syllabus)  (LessonPlayer)    (Practice)   (Profile/Stats)
```

---

## 🎛️ Unified Global State Management

Global state, user session context, and sync parameters are managed through React Context hooks:

### 1. `AppContext` (`src/context/AppContext.tsx`)
This is the master state engine of the platform, tracking user details, enrollment arrays, notifications, and core content catalogs.
- **Exposed Methods**: `signIn()`, `signUp()`, `signOut()`, `enrollInCourse()`, `updateLessonProgress()`, `addToast()`.
- **Global States**: `currentUser` (Profile), `enrollments` (List), `courses` (Array), `notifications` (Array), `isOnline` (Boolean).

### 2. `EditorContext` (`src/context/EditorContext.tsx`)
Dedicated state for lesson authors and instructors, controlling inline quiz edits, interactive document modifications, and video upload states.

### 3. `ApiKeyContext` (`src/context/ApiKeyContext.tsx`)
Secure context managing third-party tokens (such as Gemini API keys or custom YouTube API developer credentials) using local encryption algorithms.

---

## ⏱️ React Component Lifecycle Guidelines

The platform enforces strict standards to ensure high performance and prevent infinite re-renders or layout shifts:

### Effect Dependency Isolation
To prevent component recalculation loops, `useEffect` dependencies must be restricted to primitive types (strings, numbers, booleans) or memoized callbacks.

```typescript
// ❌ BAD: Triggers infinite renders because array references change on every render
useEffect(() => {
  fetchUserMetadata(userFilters);
}, [userFilters]); // userFilters is an object!

// ✅ GOOD: Isolates the primitive keys to stabilize execution
const filterString = JSON.stringify(userFilters);
useEffect(() => {
  fetchUserMetadata(JSON.parse(filterString));
}, [filterString]); // filterString is a primitive string!
```

### Resource Cleanup
All asynchronous hooks, real-time Firestore listeners, and background timers must return cleanup functions to prevent memory leaks.

```typescript
useEffect(() => {
  const q = query(collection(db, "messages"));
  
  // Create dynamic real-time listener
  const unsubscribe = onSnapshot(q, (snapshot) => {
    setMessages(snapshot.docs.map(doc => doc.data()));
  });

  // Always return cleanups
  return () => unsubscribe();
}, []);
```

---

## 🎨 Visual Design System & Branding Tokens

The application is styled with a highly polished, high-contrast palette optimized for eye safety and educational readability:

| Style Attribute | Token Value | Purpose / Application |
| :--- | :--- | :--- |
| **Primary Background** | `bg-slate-950` | Primary dark slate background for deep readability |
| **Accent Gold** | `text-[#fbbf24]` / `text-amber-400` | Accents study streaks, ratings, and time indicators |
| **Accent Green** | `text-[#34d399]` / `text-emerald-400` | Reflects progress completions and passing scores |
| **Typography Display** | `font-sans tracking-tight` | Header nodes, main titles, and lesson details |
| **Typography Data** | `font-mono text-xs` | Numerical tables, coordinates, timestamps, and schemas |

All layouts incorporate fluid scaling grids (`grid-cols-1 md:grid-cols-2 lg:grid-cols-4`) to ensure identical layouts on phone screens, tablets, and wide monitor configurations.

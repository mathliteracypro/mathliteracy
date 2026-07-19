# 📽️ Application Page Guide: Interactive Study Hub

The **Interactive Study Hub** provides students with a fully fledged, premium classroom experience. It features a custom video player loaded with bookmarks, lesson transcripts, dynamic formula overlays, and an automated study playlist tracker.

---

## 📸 Interface Preview
![Interactive Study Workspace Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Interactive+Study+Hub)
*Live UI available at: `https://mathlit.free.nf/study`*

---

## 💡 Page Purpose
The Study Hub minimizes learning friction by combining video lessons (synced with the South African CAPS curriculum) alongside relevant practice exercises, download links, and instant question submission panels.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `LessonPlayer` | `src/pages/LessonPlayer.tsx` | Main responsive workspace. Hosts the active video player, content rendering sheets, notes tabs, and resources list. |
| `YouTubePlayer` | `src/components/video/YouTubePlayer.tsx` | High-fidelity wrapper for YouTube Iframe API. Emits precise event logs (play, pause, complete, time updates) to tracking services. |
| `VideoPlaylist` | `src/components/study/VideoPlaylist.tsx` | Interactive checklist of lessons in the current module. Displays completion checkmarks and estimated completion times. |
| `VideoBookmark` | `src/components/study/VideoBookmark.tsx` | Allows students to drop flag markers at specific video timestamps, complete with custom text comments for personal review. |

---

## 🔄 Data Flow

The Study hub records real-time playback logs to save user states.

```text
               [YouTube Player Playback Event]
                              │
                    Emits timeupdate (1s)
                              │
                              ▼
                [VideoProgress.tsx component]
                              │
           Writes progress every 5 seconds to:
           /users/{uid}/video_progress/{videoId}
         ┌────────────────────┴────────────────────┐
         ▼                                         ▼
   Local DB Cache                            Cloud Sync Queue
 (Immediate React UI State)                (Enqueued for server)
```

### Video State Storage Schema (TypeScript Reference)
```typescript
export interface VideoProgressState {
  id: string;             // videoId
  userId: string;
  courseId: string;
  moduleId: string;
  lessonId: string;
  secondsWatched: number;
  durationSeconds: number;
  isCompleted: boolean;
  lastAccessed: string;   // ISO string
}
```

---

## 🖱️ User Interactions

1. **Auto-Save Timestamp**: The underlying player logs playback metrics. If a user closes their browser, returning to the page will show a "Resume at MM:SS" overlay.
2. **Dynamic Bookmark Drops**: While watching a complicated math derivation (e.g. compound interest formulas), students can press `Ctrl + B` or click "Drop Bookmark" to capture that exact frame, complete with local notes.
3. **Tabbed Resources Workspace**: Below the player, students can toggle between:
   - **Lesson Notes**: Structured markdown document rendered with mathematical formulas.
   - **Interactive Q&A**: Community Q&A feed linked to the lesson ID.
   - **Quizzes**: High-performance inline forms with multiple-choice questions for instant validation.

# ✍️ Application Page Guide: Editorial Blog

The **Editorial Blog** is an integrated content page that hosts academic publications, exam preparation newsletters, formula cheat sheets, and student success articles.

---

## 📸 Interface Preview
![Editorial Blog Workspace Layout Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Blog)
*Live UI available at: `https://mathlit.free.nf/blog`*

---

## 💡 Page Purpose
Providing contextual articles bridges the gap between mechanical calculations and real-world math applications (such as inflation tracking or architectural scaling). The blog supports structured reading and discussion threads to reinforce learning outside standard lectures.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `BlogAnalytics` | `src/components/blog/BlogAnalytics.tsx` | Background metric collector logging average read durations and content scroll depth. |
| `ReadingProgress` | `src/components/blog/ReadingProgress.tsx` | Fixed sticky top progress bar animating fluidly to visually reflect scroll percentage. |
| `BlogComments` | `src/components/blog/BlogComments.tsx` | Interactive discussion board allowing verified users to comment and reply to threads. |
| `NewsletterSignup` | `src/components/blog/NewsletterSignup.tsx` | Newsletter submission card with automated e-mail validation rules. |

---

## 🔄 Data Flow

Blog articles are queried from the root `blog_posts` collection. Comments operate as subcollections.

```text
               [User Selects Blog Article]
                            │
               Increment post view counter
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
      Query BlogPost data         Query Comment Subcollection
              │                           │
              ▼                           ▼
        Render post HTML             Render list of comments
```

### Blog Post Structure (TypeScript Reference)
```typescript
export interface BlogPost {
  id: string;
  post_title: string;
  post_slug: string;
  post_excerpt: string;
  post_content: string; // Markdown / HTML contents
  featured_image: string;
  category: "Exams" | "Finance" | "Syllabus" | "Success Stories";
  author_id: string;
  author_name: string;
  reading_time_minutes: number;
  views: number;
  is_published: boolean;
  published_date: string;
}
```

---

## 🖱️ User Interactions

1. **Reading Depth Indicator**: As a student scrolls down a long exam guide, a colored bar at the very top of the window expands. Upon reaching the bottom, a "Completed Read" event is dispatched to trigger potential experience points.
2. **Comment Submission**: Users can post comments on the article. The input box uses custom regex logic to filter out inappropriate words before enqueuing the document to the collection `/blog_posts/{id}/comments`.
3. **Dynamic Sharing**: Quick action buttons allow students to copy specialized sharing links (incorporating UTM campaign parameters) to forward to classmates.

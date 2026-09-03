# 06. Blog & Newsletter Backend Documentation

This document defines backend operations for editorial content publishing, blog comment moderation queues, and newsletter subscriber mailing lists.

---

## 1. Document Schemas

### A. Blog Post Schema (`blog_posts/{postId}`)

```json
{
  "id": "post-caps-exam-tips-2026",
  "title": "Top 10 Exam Strategies for CAPS Math Literacy Paper 1 & 2",
  "slug": "top-10-caps-math-lit-exam-tips-2026",
  "summary": "Essential tips for mastering financial calculations, scale maps, and probability.",
  "contentMarkdown": "# Introduction\nPaper 1 covers Finance, Data Handling, and Probability...",
  "coverImageUrl": "https://images.unsplash.com/photo-1434030216411-0b793f4b4173",
  "authorName": "MathLiteracy Pro Editorial Board",
  "authorAvatar": "https://lh3.googleusercontent.com/...",
  "category": "Exam Prep",
  "tags": ["Exams", "Paper 1", "Paper 2", "Grade 12"],
  "status": "published",
  "viewCount": 2450,
  "commentCount": 18,
  "publishedAt": "2026-08-15T09:00:00.000Z",
  "createdAt": "2026-08-14T14:00:00.000Z"
}
```

### B. Blog Comments Schema (`blog_comments/{commentId}`)

```json
{
  "id": "comment-9912",
  "postId": "post-caps-exam-tips-2026",
  "authorUid": "user789",
  "authorName": "Nandi Khumalo",
  "commentText": "The breakdown of tax rebate calculations saved me in my preliminary exam!",
  "status": "approved",
  "createdAt": "2026-08-16T11:20:00.000Z"
}
```

### C. Newsletter Subscription Schema (`newsletters/{subscriberId}`)

```json
{
  "id": "sub-email-hash-123",
  "email": "learner@school.za",
  "subscribedAt": "2026-08-01T10:00:00.000Z",
  "status": "active"
}
```

---

## 2. Admin Operations

### Create Blog Post
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function createBlogPost(postData: any) {
  const postId = postData.id || `post-${Date.now()}`;
  const postRef = doc(db, 'blog_posts', postId);
  await setDoc(postRef, {
    ...postData,
    id: postId,
    createdAt: new Date().toISOString()
  });
}
```

### Comment Moderation (Approve / Reject / Delete)
```typescript
import { doc, updateDoc, deleteDoc } from 'firebase/firestore';

export async function moderateComment(commentId: string, status: 'approved' | 'rejected') {
  const commentRef = doc(db, 'blog_comments', commentId);
  await updateDoc(commentRef, { status });
}

export async function deleteComment(commentId: string) {
  await deleteDoc(doc(db, 'blog_comments', commentId));
}
```

# 10. Notifications & Mailer Backend Documentation

This document covers system alert dispatches, push notification records, transactional email dispatches, and email template management.

---

## 1. Document Schemas

### A. System Notification Schema (`notifications/{notificationId}`)

```json
{
  "id": "notif-9912",
  "targetUid": "all",
  "title": "New Term 3 Assignment Published!",
  "message": "Grade 11 Finance SBA investigation is now active. Due 15 September.",
  "type": "announcement",
  "link": "/tasks",
  "readBy": ["user1", "user2"],
  "createdAt": "2026-08-31T08:00:00.000Z"
}
```

### B. Sent Email Log Schema (`sent_emails/{emailId}`)

```json
{
  "id": "mail-log-5511",
  "recipient": "learner@example.co.za",
  "subject": "MathLiteracy Pro: Certificate Issued!",
  "templateId": "cert-completion-email",
  "status": "delivered",
  "dispatchedBy": "admin-uid-1",
  "dispatchedAt": "2026-08-31T10:30:00.000Z"
}
```

---

## 2. Admin Operations

### Broadcast System Notification to All Users
```typescript
import { doc, setDoc } from 'firebase/firestore';

export async function broadcastNotification(title: string, message: string, link = '/dashboard') {
  const notifId = `notif-${Date.now()}`;
  await setDoc(doc(db, 'notifications', notifId), {
    id: notifId,
    targetUid: 'all',
    title,
    message,
    type: 'broadcast',
    link,
    readBy: [],
    createdAt: new Date().toISOString()
  });
}
```

### Backend Email Dispatch Endpoint (`/api/mail/send`)

Admin developers can trigger transactional SMTP dispatches:

```typescript
const response = await fetch('/api/mail/send', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    to: 'mathliteracypro@gmail.com',
    subject: 'Monthly Performance Digest',
    templateId: 'admin-digest',
    variables: {
      adminName: 'Dr. Phumzile',
      newStudents: 140,
      tasksGraded: 420
    }
  })
});
const result = await response.json();
```

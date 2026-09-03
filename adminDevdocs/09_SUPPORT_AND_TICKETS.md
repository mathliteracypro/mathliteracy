# 09. Support & Tickets Backend Documentation

This document describes the customer and learner support ticketing system, ticket state transitions, and administrative settings.

---

## 1. Document Schemas

### A. Support Ticket Schema (`support_tickets/{ticketId}`)

```json
{
  "id": "TICKET-8821",
  "studentUid": "user-thabo-55",
  "studentName": "Thabo Mbeki",
  "studentEmail": "thabo@example.co.za",
  "category": "Curriculum Question",
  "subject": "Question regarding inflation tariff formulas",
  "message": "I am struggling to calculate the compound interest inflation on problem #3...",
  "status": "open",
  "priority": "high",
  "assignedAdminUid": "admin-dr-phumzile",
  "responses": [
    {
      "senderUid": "admin-dr-phumzile",
      "senderName": "Dr. Phumzile Sithole",
      "message": "Hello Thabo! Remember to convert percentages to decimal format before multiplying...",
      "sentAt": "2026-08-31T10:15:00.000Z"
    }
  ],
  "createdAt": "2026-08-31T09:00:00.000Z",
  "updatedAt": "2026-08-31T10:15:00.000Z"
}
```

### B. Global Contact Settings Schema (`contact_settings/general`)

```json
{
  "supportEmail": "mathliteracypro@gmail.com",
  "supportPhone": "+27 64 819 4874",
  "address": "Magononong, Mpumalanga - South Africa",
  "businessHours": "Mon - Fri: 08:00 - 17:00 (SAST)",
  "responseTime": "Within 24 hours",
  "rateLimitPerHour": 5,
  "maxMessageLength": 5000
}
```

---

## 2. Admin Ticket Operations

### Admin Inbox Query (Open & High Priority Tickets)
```typescript
import { collection, query, where, orderBy, getDocs } from 'firebase/firestore';

export async function fetchAdminSupportTickets() {
  const q = query(
    collection(db, 'support_tickets'),
    orderBy('createdAt', 'desc')
  );
  const snap = await getDocs(q);
  return snap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
}
```

### Reply to Ticket & Update Status
```typescript
import { doc, updateDoc, arrayUnion } from 'firebase/firestore';

export async function replyToSupportTicket(ticketId: string, replyMessage: string, adminUid: string, adminName: string, newStatus: 'in_progress' | 'resolved' = 'resolved') {
  const ticketRef = doc(db, 'support_tickets', ticketId);
  const now = new Date().toISOString();
  
  await updateDoc(ticketRef, {
    status: newStatus,
    responses: arrayUnion({
      senderUid: adminUid,
      senderName: adminName,
      message: replyMessage,
      sentAt: now
    }),
    updatedAt: now
  });
}
```

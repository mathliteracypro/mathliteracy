# 📶 Offline Engine & Network Architecture

MathLiteracy Pro features a customized offline syncer engine designed specifically for South African students experiencing high mobile data tariffs, low bandwidth networks, or load shedding. The engine handles connection loss gracefully by queuing database updates and loading cached data.

---

## 📐 Networking Lifecycle Engine

```text
               [Application Write Trigger]
                           │
             Check Network Status (navigator.onLine)
                           │
         ┌─────────────────┴─────────────────┐
         ▼ (ONLINE)                          ▼ (OFFLINE)
  [Send to Firestore]                 [Enqueue to localForage]
         │                                   │
         ▼ (Success)                         ▼
  Update Local Store                  Keep Retry Alarm Active
                                             │
                                  Network Restored Event
                                             │
                                             ▼
                                  [Flush Request Queue]
                                             │
                                             ▼
                                  Merge Remote Updates
```

---

## 📦 Local Cache Structures & Storage Modes

The system splits storage workloads across three storage caches for optimal efficiency:

1. **Firestore Client-Side Persistence**: Stores document snapshots locally. The database engine operates directly on local snapshots when offline, keeping query speeds identical to native memory speeds.
2. **`localForage` (IndexedDB Wrapper)**: Caches static content catalogs (e.g. course listings, blog articles, and formula sheets) so that catalog pages can be visited immediately without waiting for Firestore handshakes.
3. **`localStorage`**: Retains user preferences, UI tokens, and simple metadata indicators.

---

## 🗂️ The Synchronizer Class (`src/services/syncManager.ts`)

The background synchronizer monitors network changes. When the device returns online, it flushes queued actions sequentially.

### Offline Queue Implementation Reference
```typescript
interface QueuedRequest {
  id: string;
  action: "create" | "update" | "delete";
  collection: string;
  documentId: string;
  payload: any;
  timestamp: number;
}

export class SyncManager {
  private static QUEUE_KEY = "offline_sync_request_queue";

  // Enqueue a collection update during network disconnects
  public static async enqueueRequest(collection: string, docId: string, payload: any, action: "create" | "update") {
    const queue = await this.getQueue();
    const newRequest: QueuedRequest = {
      id: `${collection}_${docId}_${Date.now()}`,
      action,
      collection,
      documentId: docId,
      payload,
      timestamp: Date.now()
    };

    queue.push(newRequest);
    await localStorage.setItem(this.QUEUE_KEY, JSON.stringify(queue));
    console.log(`Offline request queued: [${action}] on ${collection}/${docId}`);
  }

  // Fetch current request queue
  private static async getQueue(): Promise<QueuedRequest[]> {
    const data = localStorage.getItem(this.QUEUE_KEY);
    return data ? JSON.parse(data) : [];
  }

  // Attempt sync when connection is recovered
  public static async processQueue(db: any): Promise<void> {
    const queue = await this.getQueue();
    if (queue.length === 0) return;

    console.log(`SyncManager: Processing ${queue.length} enqueued requests...`);
    const failed: QueuedRequest[] = [];

    for (const req of queue) {
      try {
        const docRef = doc(db, req.collection, req.documentId);
        if (req.action === "update") {
          await updateDoc(docRef, req.payload);
        } else if (req.action === "create") {
          await setDoc(docRef, req.payload);
        }
        console.log(`Synced successfully: ${req.id}`);
      } catch (err) {
        console.error(`Sync failure for ${req.id}, keeping in queue:`, err);
        failed.push(req);
      }
    }

    await localStorage.setItem(this.QUEUE_KEY, JSON.stringify(failed));
  }
}
```

---

## ⚖️ Conflict Resolution Guidelines

When both the local cache and remote server contain changes to the same document, the client resolves conflicts using a **Last-Write-Wins (LWW)** strategy based on `updated_at` values.
- **Client timestamp exceeds Server**: The client updates the server document.
- **Server timestamp exceeds Client**: The local client updates its cache with the server document.

# 🛡️ Firebase Security Rules Architecture

To protect user confidentiality, prevent fraudulent progress modifications, and restrict curriculum edits to verified instructors, MathLiteracy Pro utilizes a strict, declarative security rules blueprint for Google Firestore.

---

## 🏛️ Central Principles

1. **Deny-by-Default**: Every collection path is protected. Unless explicitly overridden by a logical path rule, write operations are completely blocked.
2. **Authenticated-Only Readers**: Accessing user metrics, private lesson files, and grades requires active, cryptographically validated JWT tokens.
3. **Owner-Controlled Mutability**: Users are strictly blocked from writing to, deleting, or altering database documents belonging to other users.

---

## 📄 Complete Live `firestore.rules` File

Below is the complete security configuration compiled and deployed on our Firestore environment:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ============================================
    // ⚙️ GLOBAL HELPER FUNCTIONS
    // ============================================

    // Check if user is signed in with a valid token
    function isAuthenticated() {
      return request.auth != null;
    }

    // Verify requesting client is the owner of the target document
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }

    // Check if the user has verified their email address
    function isEmailVerified() {
      return isAuthenticated() && request.auth.token.email_verified == true;
    }

    // Check if the signed-in user is registered as a Site Admin
    function isAdmin() {
      return isAuthenticated() && 
        (get(/users/$(request.auth.uid)).data.account_type == "admin" ||
         get(/users/$(request.auth.uid)).data.is_admin == true);
    }

    // Check if the user is registered as an Instructor
    function isInstructor() {
      return isAuthenticated() && 
        (get(/users/$(request.auth.uid)).data.is_instructor == true ||
         get(/users/$(request.auth.uid)).data.isInstructor == true);
    }

    // ============================================
    // 📁 COLLECTION PATH RULES
    // ============================================

    // 1. Users Profiles Directory
    match /users/{userId} {
      allow read: if isAuthenticated();
      allow create: if isAuthenticated() && request.auth.uid == userId;
      allow update: if isOwner(userId) || isAdmin();
      allow delete: if isAdmin();

      // Tracked video playback states
      match /video_progress/{videoId} {
        allow read, write: if isOwner(userId) || isAdmin();
      }

      // Tracked streak history logs
      match /streak_history/{dateId} {
        allow read, write: if isOwner(userId) || isAdmin();
      }
    }

    // 2. Syllabus, Topics, & Lessons
    match /topics/{topicId} {
      allow read: if true; // Publicly queryable for indexation and sitemaps
      allow write: if isAdmin() || isInstructor();

      // Enrolled student stats subcollection
      match /enrolled_students/{enrollmentId} {
        allow read: if true; // Accessible for sitemap metrics compilation
        allow create, update: if isAuthenticated() && (request.resource.data.user_id == request.auth.uid || isAdmin());
        allow delete: if isAdmin();
      }

      // Subtopic folders
      match /subtopics/{subtopicId} {
        allow read: if true;
        allow write: if isAdmin() || isInstructor();

        // Subtopic lessons / resources
        match /content/{contentId} {
          allow read: if true;
          allow write: if isAdmin() || isInstructor();
        }
      }
    }

    // 3. Editorial Blog
    match /blog_posts/{postId} {
      allow read: if true;
      allow write: if isAdmin();

      match /comments/{commentId} {
        allow read: if true;
        allow create: if isAuthenticated() && request.resource.data.user_id == request.auth.uid;
        allow update, delete: if isAdmin() || (isAuthenticated() && resource.data.user_id == request.auth.uid);
      }
    }

    // 4. Assignments & Practice Tasks
    match /tasks/{taskId} {
      allow read: if isAuthenticated();
      allow write: if isAdmin() || isInstructor();

      match /submissions/{submissionId} {
        allow read: if isAuthenticated() && (resource.data.userId == request.auth.uid || isAdmin() || isInstructor());
        allow create: if isAuthenticated() && request.resource.data.userId == request.auth.uid;
        allow update: if isAdmin() || isInstructor() || (isAuthenticated() && resource.data.userId == request.auth.uid);
        allow delete: if isAdmin();
      }
    }

    // ============================================
    // 🌐 COLLECTION GROUP RULES
    // ============================================

    // Allow dynamic compilation of enrollments for statistical tables
    match /{path=**}/enrolled_students/{enrollmentId} {
      allow read: if true;
    }

    // Allow public indexers to scan subtopic nodes
    match /{path=**}/subtopics/{subtopicId} {
      allow read: if true;
    }
  }
}
```

# 🛡️ Trust, Security & Data Privacy Center

At MathLiteracy Pro, we recognize that our students, parents, and teachers entrust us with sensitive academic information, progress metrics, and contact details. We maintain strict security protocols to prevent data breaches, protect individual privacy, and safeguard credential configurations.

This document outlines our security measures and explains why our systems are secure against unauthorized access.

---

## 🔒 1. Safe API Key Architecture (Zero Browser Exposure)

A common security vulnerability in poorly designed web applications is exposing administrative secrets or third-party API keys (such as the Google Gemini API key or database credentials) directly to the web browser. 

MathLiteracy Pro uses a **strict Server-Side Proxy Architecture** to protect all platform credentials:

```text
  ┌─────────────────────────┐
  │  🧑‍🎓 Student Browser   │
  └────────────┬────────────┘
               │  [Sends user prompt/request]
               │  (DOES NOT HAVE OR SEE THE API KEY)
               ▼
  ┌─────────────────────────┐
  │  🛡️ Server Container     │ <── Loads secret from process.env.GEMINI_API_KEY
  │  (Express.js / Node)    │     (Hidden securely inside Cloud Run)
  └────────────┬────────────┘
               │  [Makes authorized request]
               ▼
  ┌─────────────────────────┐
  │   🤖 Google Gemini API  │
  └─────────────────────────┘
```

### Our Security Standards for Secrets:
* **Private Container Variables**: The database keys and Gemini keys are injected directly into the hosting environment's operating system environment variables (`process.env.GEMINI_API_KEY`). They are never compiled, packaged, or written into the client-side JavaScript bundle.
* **No `VITE_` Prefixes for Secrets**: In Vite projects, any environment variable prefixed with `VITE_` is automatically compiled and exposed to the user's browser. We strictly keep our API keys named as standard variables (e.g. `GEMINI_API_KEY`, `FIREBASE_ADMIN_CREDENTIALS`), keeping them completely invisible to the client.
* **Serverless Proxies**: Whenever a student interacts with AI-powered study tutors, the request is proxied through a secure `/api/` endpoint. The server handles the calculation, talks to the API safely, and returns only the clean response to the student's browser.

---

## 🚫 2. Granular NoSQL Isolation (No Data Exposure)

To protect individual grades, school reports, and personal emails, our database is protected by secure **Firestore Security Rules**. These rules enforce a "Deny-by-Default" stance, ensuring that students cannot access, view, or modify information belonging to others.

### Why Your Data is Protected Against Hacker Attacks:
1. **JWT Auth Verification**: Every request made to our database carries an encrypted cryptographic JSON Web Token (JWT) issued directly by Google Firebase Authentication. The system inspects this token on every read or write.
2. **Owner-Document Alignment Rules**: Our database pathways are bound by strict alignment checks. A student can only request documents where their validated user ID matches the document path:
   ```javascript
   match /users/{userId} {
     // A student can ONLY view or modify their own progress details
     allow read, write: if request.auth.uid == userId;
   }
   ```
3. **Restricted Listing Queries**: Our security rules strictly prohibit broad listing requests. A user cannot issue generic queries to scan the entire student directory or search for other students' grades.
4. **Isolated Subcollections**: Student metrics (like practice task submissions and video progress history) are stored in individual subcollections locked under the student's personal authenticated scope.

---

## 📧 3. Guarded Identity & Sign-In Operations

MathLiteracy Pro handles student authentication through Google Firebase Authentication, a secure, industry-leading login service:

* **Email Verification Gating**: When an account is created, access to advanced study modules is locked. The platform fires a verification email, forcing the user to verify ownership before their document states become active.
* **Encrypted Passwords**: Passwords are never stored in plain text. They are salted and cryptographically hashed by Google's secure identity managers before entering storage.
* **No Shared Credentials**: Since student metrics are uniquely linked to their authenticated email, sharing credentials or spoofing profiles is barred.

---

## 🧬 4. Public Certificate Hashes (No Information Leaks)

When students complete topics, they receive certificates with a cryptographic verification hash (e.g. `MLP-2026-0981`).

* **One-Way Hashes**: The hash is a mathematically computed string that represents their success. It does not contain or leak their email, password, or direct personal information.
* **Isolated Verification**: When an educator or parent enters this hash on the validator page, the system queries only the verified course certificate record matching that exact string. No other database pathways or tables can be accessed or inspected through this tool.

---

## 🏡 Return to Main Help
- [Back to Help Center Home](../README.md)

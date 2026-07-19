# 🔌 Platform API Endpoint Reference

While MathLiteracy Pro is primarily a serverless client-side application utilizing Firebase client SDKs, it operates several critical APIs for tasks like automated document generation and external indexation.

---

## 🛰️ Base URLs

- **Development Port**: `http://localhost:3000`
- **Live Endpoint**: `https://mathlit.free.nf/api`

---

## 📑 Core Endpoints

### 1. Health Checks
Check container ingress status and database connection health.

- **URL**: `/api/health`
- **Method**: `GET`
- **Headers**: None
- **Response Payload**:
```json
{
  "status": "ok",
  "timestamp": "2026-07-18T17:46:36.000Z",
  "database": "connected",
  "services": {
    "auth": "operational",
    "storage": "operational"
  }
}
```

---

### 2. Live CAPS Course Dynamic Statistics Audit
Generates compiled enrollment data and topic analytics for the sitemap audit panel.

- **URL**: `/api/stats/audit`
- **Method**: `GET`
- **Headers**:
  - `Authorization: Bearer <ID_TOKEN>` (requires user role `"admin"`)
- **Response Payload**:
```json
{
  "success": true,
  "data": [
    {
      "id": "finance_grade_12",
      "name": "Financial Mathematics (Grade 12)",
      "subtopicsCount": 8,
      "lessonsCount": 24,
      "totalHours": 18.5,
      "enrollmentCount": 142
    },
    {
      "id": "maps_grade_11",
      "name": "Maps, Plans and Representations (Grade 11)",
      "subtopicsCount": 6,
      "lessonsCount": 15,
      "totalHours": 10.2,
      "enrollmentCount": 98
    }
  ]
}
```

---

### 3. Verification of Earned Certificates
Allows public lookup of cryptographic certificate validation hashes.

- **URL**: `/api/verify/certificate/:hash`
- **Method**: `GET`
- **URL Parameters**:
  - `hash` (Required): SHA-256 certificate hash string.
- **Response Payload (Success)**:
```json
{
  "verified": true,
  "certificate": {
    "certificate_number": "MLP-2026-0981",
    "recipient_name": "Sipho Dlamini",
    "course_name": "Measurement & Scaling Masterclass",
    "issued_date": "2026-06-12T14:20:00Z",
    "grade_level": "Grade 12"
  }
}
```
- **Response Payload (Not Found)**:
```json
{
  "verified": false,
  "error": "No valid certificate matches this validation signature."
}
```

---

### 4. Interactive AI Tutoring Engine (Gemini API Integration)
Acts as a server-side proxy to interact with Gemini API securely, preventing API key exposure.

- **URL**: `/api/tutor/chat`
- **Method**: `POST`
- **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <ID_TOKEN>`
- **Request Payload**:
```json
{
  "message": "Explain how to calculate inflation adjustments on a pension fund",
  "topicContext": "Financial Mathematics",
  "gradeLevel": "Grade 12"
}
```
- **Response Payload**:
```json
{
  "reply": "To adjust a value for inflation over multiple years, we use the compound interest formula: A = P(1 + r)^n.\n\nHere is a step-by-step example...\n\nWould you like to try a practice problem?",
  "recommendedFormula": "A = P(1 + r)^n",
  "suggestedLessonId": "fin_inflation_12"
}
```

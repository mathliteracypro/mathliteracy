# 08. Certificates Management Backend Documentation

This document defines the issuance, verification hashing, and revocation workflows for **Mathematical Literacy CAPS Topic Completion Certificates**.

---

## 1. Document Schema (`certificates/{certificateId}`)

```json
{
  "id": "CERT-2026-FIN-9921",
  "studentUid": "user-sipho-123",
  "studentName": "Sipho Nkosi",
  "topicId": "finance-tariffs-g11",
  "topicTitle": "Finance: Municipal Tariffs & Taxes",
  "grade": 11,
  "issueDate": "2026-08-31T10:30:00.000Z",
  "verificationHash": "a9f8b2c1d3e4f5a6b7c8d9e0f1a2b3c4",
  "issuer": "MathLiteracy Pro National Academic Board",
  "scoreAverage": 92.5,
  "status": "valid",
  "pdfDownloadUrl": "https://storage.googleapis.com/.../CERT-2026-FIN-9921.pdf"
}
```

---

## 2. Certificate Issuance Logic

A certificate is automatically or manually issued when a learner completes all subtopics and lessons within a topic with a minimum pass score (e.g. >= 50%).

```typescript
import { doc, setDoc, getDoc } from 'firebase/firestore';

export async function issueCertificate(studentUid: string, studentName: string, topicId: string, topicTitle: string, grade: number, scoreAverage: number) {
  const certId = `CERT-${new Date().getFullYear()}-${topicId.substring(0, 4).toUpperCase()}-${Math.floor(1000 + Math.random() * 9000)}`;
  const verificationHash = String(Date.now()) + studentUid + topicId;
  
  const certData = {
    id: certId,
    studentUid,
    studentName,
    topicId,
    topicTitle,
    grade,
    issueDate: new Date().toISOString(),
    verificationHash,
    issuer: 'MathLiteracy Pro Academic Board',
    scoreAverage,
    status: 'valid'
  };

  await setDoc(doc(db, 'certificates', certId), certData);
  return certData;
}
```

---

## 3. Public Certificate Verification Query

```typescript
export async function verifyCertificate(certificateId: string) {
  const certSnap = await getDoc(doc(db, 'certificates', certificateId));
  if (!certSnap.exists()) {
    return { valid: false, message: 'Certificate record not found.' };
  }
  const cert = certSnap.data();
  if (cert.status !== 'valid') {
    return { valid: false, message: 'This certificate has been revoked or invalidated.' };
  }
  return { valid: true, certificate: cert };
}
```

---

## 4. Admin Revocation Action

```typescript
export async function revokeCertificate(certificateId: string, reason: string) {
  await updateDoc(doc(db, 'certificates', certificateId), {
    status: 'revoked',
    revokedReason: reason,
    revokedAt: new Date().toISOString()
  });
}
```

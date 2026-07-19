# 🔐 Application Page Guide: Authentication & Gateways

The **Authentication & Gateways** page manages registration, credential validations, onboarding setup, and login checks across the MathLiteracy Pro network.

---

## 📸 Interface Preview
![Auth Page Workspace Layout Placeholder](https://via.placeholder.com/1200x600.png?text=MathLiteracy+Pro+Secure+Login)
*Live UI available at: `https://mathlit.free.nf/auth`*

---

## 💡 Page Purpose
Ensuring data privacy, tracking student-specific curriculum stats, and preserving study logs requires a secure authentication system. This page handles student registration, password resets, role designations, and verified email requirements.

---

## 🧱 Key Components

| Component Name | File Path | Description |
| :--- | :--- | :--- |
| `Auth` | `src/pages/Auth.tsx` | Main auth terminal containing beautiful layout panels to switch between "Login" and "Create Account". |
| `Onboard` | `src/pages/Onboard.tsx` | Onboarding flow guiding newly registered students to select their Grade (10, 11, or 12), school, and learning priorities. |
| `Verify` | `src/pages/Verify.tsx` | Informative visual block blocking access for unverified emails and providing triggers to resend validation links. |
| `ProfileApiKeySection` | `src/components/profile/ProfileApiKeySection.tsx` | Displays and manages authorization key strings for API development access. |

---

## 🔄 Data Flow

The authentication suite connects directly to Firebase Auth, which propagates records to Firestore:

```text
               [User Submits Email & Password]
                             │
               Firebase Auth signs up user
                             │
                             ▼
         [Fires Cloud Trigger: Creates Profile]
             Saves to Firestore: /users/{uid}
                             │
                             ▼
              Redirects to Onboarding Wizard
                             │
                             ▼
               Sets isOnboarded = true on completion
```

### Complete Auth Schema Reference
```typescript
export interface FirebaseUserRecord {
  uid: string;
  email: string;
  emailVerified: boolean;
  displayName: string;
  photoURL: string | null;
  phoneNumber: string | null;
}
```

---

## 🖱️ User Interactions

1. **Animated Password Strength Metrics**: While typing a new password, an interactive meter fluidly transitions colors from red (weak) to green (strong) using CSS transitions.
2. **Dynamic Email Verification Gating**: When a user registers, they are automatically placed in a "Pending Verification" state. The interface displays a clean countdown letting them resend the link once every 60 seconds.
3. **Interactive Onboarding Selector**: Newly joined users are prompted through a beautiful screen with large card grids to select their school grade level, helping the system pre-configure relevant math study recommendation feeds on the dashboard.

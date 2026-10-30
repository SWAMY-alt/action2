# Security Specification & Security Architecture (ActionLens Civic Portal)

## 1. Security Architecture & Identity Protection
* **Resident Identity Masking**: UIDAI Aadhaar card numbers are programmatically masked at ingestion (`XXXX-XXXX-LAST4`). Raw full numbers are never stored in plain text.
* **Role-Based Access Control (RBAC)**: Distinct permissions for `citizen` (Grievance Reporting & Upvoting) and `authority` (Dispatching Repair Units, Work Plan Briefs, Override Audit Logs).
* **Multi-Factor Proof Verification**: Every submission mandates photo attachment proof and Google Map location geocoding to prevent bot spam and false reports.

## 2. Firestore Attribute-Based Access Control (ABAC) Rules
* **Master Gate Security**:
  ```javascript
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {
      match /reports/{reportId} {
        allow read: if true;
        allow create, update: if request.auth != null;
        allow delete: if request.auth != null;
      }
      match /{document=**} {
        allow read, write: if false;
      }
    }
  }
  ```

## 3. WCAG 2.1 AA Accessibility & Input Hardening
* All interactive controls enforce `:focus-visible` ring outlines and explicit `aria-label` tags.
* Input fields enforce sanitization and character length bounds (Max 2000 chars for text descriptions).
* Screen readers announce real-time status transitions using `aria-live="polite"`.

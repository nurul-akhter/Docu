# Guardian App — Login & Add Child Flow

## Overview

### Guardian Types

**Key Guardian**

- Designated by the school as the primary contact for a child.
- Authenticates via email + password.
- Adds a child using student ID + date of birth, verified by OTP sent to their registered phone.

**Additional Guardian**

- Self-registers in the app (up to 3 per child).
- Without key guardian registration, child cannot be added by additional guardian.
- Authenticates via email + password.
- Adds a child using student ID + date of birth; requires key guardian in-app approval before the child is linked.

### Core Rules

| Rule                   | Detail                                                                                                                                                 |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Authentication         | Email + password for all guardians. No OTP on login.                                                                                                   |
| Key Guardian Detection | Determined by matching registered email with student master's key guardian email during add-child flow.                                                |
| OTP                    | Only during add-child flow if detected as key guardian (email matches student master key guardian email). OTP sent to key guardian's registered phone. |
| In-App Approval        | For additional guardians (email does not match student master key guardian email). Key guardian must approve access to child.                          |
| Child access           | No automatic inheritance. Every guardian must explicitly add each child.                                                                               |
| Device binding         | One account = one active device. New device login locks the previous device, all linked child will reload to the new device.                           |
| Password reset         | No impact on local data immediately.                                                                                                                   |

---

## Flow 1: Guardian Registration (Key or Additional)

Both guardian types register the same way. The distinction is determined during the add-child step by matching the registered email address against the student master's key guardian email.

```
Guardian opens app
    ↓
Taps "Register"
    ↓
Enters:
  - Full name
  - Email
  - Password
  - Confirm password
    ↓
Submit
    ↓
Backend creates account
    ↓
"Add Your First Child" (Flow 4)
```

**Validations**:

- Email: valid format, not already registered
- Password: min 8 chars, at least 1 uppercase, 1 lowercase, 1 number

---

## Flow 2: Login (All Guardians — Returning)

```
Guardian opens app
    ↓
Enters email + password
    ↓
Backend validates credentials
    ↓
If account is revoked:
  → Show "Your access has been revoked. Contact the key guardian."
    ↓
Dashboard loads with last active child (cached data)
Background sync triggers automatically
```

**No OTP on login.** OTP only occurs during add-child (Flow 4 / Flow 5).

**Background Sync** (non-blocking):

- Dashboard shows immediately with cached data
- Sync status: "Syncing..." → "Updated just now"
- If sync detects revocation: immediate forced logout

---

## Flow 3: New Device Login

**Trigger**: Guardian logs in from an unrecognised device.

```
Guardian enters credentials
    ↓
Backend detects device_id not matching registered device
    ↓
┌─────────────────────────────────────┐
│  New Device Detected                │
│                                     │
│  Logging in here will lock your     │
│  previous device immediately.       │
│                                     │
│  [Continue]         [Cancel]        │
└─────────────────────────────────────┘
    ↓
On Continue:
  - New  device information update
  - FCM token updated
  - Full initial sync triggered (blocking, with progress bar — same as Flow 4 Step 3)
```

---

## Flow 4: Key Guardian — First Child Onboarding

**Trigger**: Key guardian completes registration with no children linked.

### Step 1: Enter Student Details

```
┌─────────────────────────────────────┐
│   Add Your First Child              │
│                                     │
│   Student ID                        │
│   [________________________]        │
│   (shared by school via SMS/email)  │
│                                     │
│   Date of Birth                     │
│   [________________________]        │
│                                     │
│   Relationship to child             │
│   [________________________]        │
│                                     │
│   [Submit]                          │
└─────────────────────────────────────┘
```

**Validations**:

- Student ID exists in school system
- Date of birth matches school records
- **Registered email matches student master's key guardian email** (determines if user is key guardian)
- Max 3 wrong attempts → 5-minute lockout

**On Invalid**:

```
"Student ID and Date of Birth do not match. Please check and try again."
[Try Again]
```

---

### Step 2: OTP Verification (Key Guardian Only)

**On valid student details**, if registered email matches the student master's key guardian email, backend:

1. Looks up key guardian's phone from student master using the student ID
2. Generates 6-digit OTP
3. Sends OTP to key guardian's registered phone

**If email does not match key guardian email**: proceed to in-app approval flow (Flow 5) instead.

```
┌─────────────────────────────────────┐
│   Verify Your Identity              │
│                                     │
│   OTP sent to: +880 1234 ****56     │
│   (last 2 digits shown)             │
│                                     │
│   [_ _ _ _ _ _]                     │
│   Enter the 6-digit code            │
│                                     │
│   [Verify]                          │
│                                     │
│   Didn't receive? [Resend OTP]      │
│   (available after 30 seconds)      │
└─────────────────────────────────────┘
```

**OTP Rules**:

- 6-digit numeric
- Expiry: 10 minutes
- Max 3 wrong attempts → must resend
- Resend cooldown: 30 seconds, max 3 resends per attempt

**On Valid OTP** → proceed to Step 3

**On Invalid OTP**: "Incorrect code. Try again." (3 failures: "Too many attempts. Resend the code.")

---

### Step 3: Device Registration & Initial Sync

**On OTP success**, backend links child to guardian:

```sql
INSERT INTO guardian_children (guardian_id, child_id, school_id, relationship, linked_at)
VALUES (guardian_id, child_id, school_id, 'Father', NOW())
```

App shows sync progress (blocking):

```
┌─────────────────────────────────────┐
│   Setting Up Your Account...        │
│                                     │
│   [████████░░] 45%                  │
│                                     │
│   - Registering device...           │
│   - Downloading school data...      │
│   - Downloading student data...     │
│   - Configuring school theme...     │
│   - Finalizing...                   │
│                                     │
│   Please wait. Do not close the app.│
└─────────────────────────────────────┘
```

**App Action Sequence**:

```
1. Capture device info:
   - device_id (generated and stored locally)
   - fcm_token (Firebase Cloud Messaging)
   - device_brand (e.g., "Samsung")
   - device_model (e.g., "Galaxy S21")
   - os_version (e.g., "Android 13")

2. Register device:
   POST /api/auth/device-register
   {
     "guardian_id": "uuid",
     "device_id": "unique-device-id",
     "fcm_token": "token",
     "brand": "Samsung",
     "model": "Galaxy S21",
     "os_version": "Android 13"
   }
   Response: { "status": "registered", "device_id": "uuid" }

3. Download initial data:
   POST /api/sync/initial-login
   {
     "school_id": "uuid",
     "guardian_id": "uuid",
     "child_id": "uuid",
     "device_id": "uuid"
   }
   Response: { "school_info": {...}, "student_profile": {...}, "attendance": [...], ... }

4. Create local SQLite DB:
   - Create all tables
   - Insert downloaded data
   - Set last_sync_datetime = NOW()

5. Apply school theme:
   - Download logo, colors, fonts
   - Store locally and apply to UI

6. Subscribe to FCM topics:
   - "school_{school_id}"
   - "child_{child_id}"
   - "guardian_{guardian_id}"

7. Show: "Setup complete ✓"
```

**On Sync Failure**:

```
┌─────────────────────────────────────┐
│   Sync Failed                       │
│                                     │
│   Check your internet connection    │
│   and try again.                    │
│                                     │
│   [Retry]        [Contact Support]  │
└─────────────────────────────────────┘

- Retry resumes from Step 3 (device already registered)
- After 3 consecutive failures: allow dashboard in offline mode,
  sync retried automatically when connection is restored
```

---

## Flow 5: Additional Guardian — Add Child Request

**Trigger**: Additional guardian taps [+ Add Child].

### Step 1: Enter Student Details

Same input screen as Flow 4 Step 1 (student ID, date of birth, relationship).

**Validations**:

- Student ID exists in school system
- Date of birth matches school records
- Not allowed if already 3 additional guardians registered (including pending approvals)
- Max 3 wrong attempts → 5-minute lockout

### Step 2: In-App Approval Request (No OTP)

**On valid student details**, backend:

1. Looks up key guardian from student master using the student ID
2. Creates a pending approval request
3. Sends push notification to key guardian

```
Key guardian receives:
"[Guardian Name] is requesting access to [Child Name]. Approve or Deny."
```

Additional guardian sees:

```
┌─────────────────────────────────────┐
│   Request Sent                      │
│                                     │
│   Your request to access            │
│   [Child Name] has been sent to     │
│   the key guardian for approval.    │
│                                     │
│   You will be notified once         │
│   approved or denied.               │
│                                     │
│   Request expires in 48 hours.      │
└─────────────────────────────────────┘
```

### Step 3: Key Guardian Responds

Key guardian opens the approval request (from notification or Requests section):

```
┌─────────────────────────────────────┐
│   Guardian Access Request           │
│                                     │
│   Name:         [Guardian Name]     │
│   Email:        [guardian@email]    │
│   Relationship: [e.g., Uncle]       │
│   Child:        [Child Name]        │
│                                     │
│   [Approve]          [Deny]         │
└─────────────────────────────────────┘
```

**On Approve**:

```
Backend:
  - Links child to additional guardian
  - Triggers initial sync for additional guardian's device
  - Sends push to additional guardian: "Access approved. [Child Name] added to your account."
```

**On Deny**:

```
Backend:
  - Marks request as denied
  - Sends push to additional guardian: "Your request to access [Child Name] was denied."
  - Guardian may re-submit after 24 hours
```

**Request expiry (48h with no response)**:

```
Push to additional guardian: "Your request expired. Please re-submit."
Guardian may re-submit immediately.
```

---

## Flow 6: Adding More Children (Any Guardian)

**Trigger**: Logged-in guardian taps [+ Add Child].

- **If registered email matches child's key guardian email**: follows Flow 4 Steps 1–3 (student ID + DOB → email validation → OTP → sync)
- **If registered email does not match key guardian email**: follows Flow 5 (student ID + DOB → key guardian in-app approval → sync)

**Rules**:

- Full sync must complete before adding another child
- Session stays alive — no logout required
- Up to 3 additional guardians allowed per child (enforced by backend, considering already added + pending approvals)

---

## Flow 7: Revoking Additional Guardian Access

**Trigger**: Key guardian revokes an additional guardian from the Manage Guardians screen.

```
Key guardian selects additional guardian → taps [Revoke Access]
    ↓
Confirmation: "Remove [Name]? They will lose access to all linked children immediately."
    ↓
On Confirm:
  Backend:
    - Marks guardian status → revoked
    - Flags all linked child records for that guardian as removed
    ↓
On next sync on additional guardian's device:
  - Sync response includes revocation flag
  - App forces immediate logout
  - Local data cleared
  - Redirected to login screen with:
    "Your access has been revoked. Contact the key guardian."
```

---

## Flow 8: Forgot Password

```
Login screen → taps "Forgot password?"
    ↓
Enters registered email address
    ↓
Backend sends password reset link to email
    ↓
┌─────────────────────────────────────┐
│   Check Your Email                  │
│                                     │
│   A reset link has been sent to     │
│   [email]. Link expires in 30 min.  │
│                                     │
│   [Resend Email]                    │
│   (available after 60 seconds)      │
└─────────────────────────────────────┘
    ↓
Guardian taps link in email → opens app or browser
    ↓
Enters new password + confirm password
    ↓
Password updated → all active sessions invalidated → redirected to login
```

**Rules**:

- Reset link expires after 30 minutes
- Link is single-use
- All active sessions (all devices) are invalidated immediately on password reset

---

## Error Scenarios

### Registration

| Error                    | Message                                           | Action            |
| ------------------------ | ------------------------------------------------- | ----------------- |
| Email already registered | "An account with this email already exists"       | Offer login link  |
| Invalid email format     | "Enter a valid email address"                     | Inline validation |
| Weak password            | "Password must be 8+ chars, mixed case, 1 number" | Inline validation |

### Login

| Error             | Message                                                   | Action          |
| ----------------- | --------------------------------------------------------- | --------------- |
| Wrong credentials | "Incorrect email or password"                             | Allow retry     |
| Account locked    | "Account locked. Try again in 15 minutes."                | Countdown shown |
| Access revoked    | "Your access has been revoked. Contact the key guardian." | No login        |

### Add Child (Student ID / DOB)

| Error                     | Message                                        | Action        |
| ------------------------- | ---------------------------------------------- | ------------- |
| Student ID + DOB mismatch | "Student ID and Date of Birth do not match."   | Retry (max 3) |
| Child already linked      | "This child is already added to your account." | —             |
| 3 failed attempts         | "Too many attempts. Try again in 5 minutes."   | Countdown     |

### OTP (Key Guardian — Add Child)

| Error        | Message                               | Action             |
| ------------ | ------------------------------------- | ------------------ |
| Wrong OTP    | "Incorrect code. Try again."          | 3 attempts allowed |
| OTP expired  | "Code expired. Request a new one."    | [Resend] button    |
| Max attempts | "Too many attempts. Resend the code." | 30-sec cooldown    |

### Additional Guardian Approval

| Error                      | Message                                          | Action                   |
| -------------------------- | ------------------------------------------------ | ------------------------ |
| Key guardian not reachable | "Could not send request. Try again later."       | [Retry]                  |
| Request already pending    | "A request for this child is already pending."   | Show waiting screen      |
| Request denied             | "Your request was denied."                       | Re-submit after 24 hours |
| Request expired (48h)      | "Your request expired. Please re-submit."        | Immediate re-submit      |
| Guardian limit reached     | "This child already has 3 additional guardians." | No further requests      |

### Sync

| Error           | Message                                   | Action  |
| --------------- | ----------------------------------------- | ------- |
| Network timeout | "Connection failed. Check your internet." | [Retry] |
| Server error    | "Server error. Try again soon."           | [Retry] |
| Partial failure | "Some data failed to sync."               | [Retry] |

---

## Data Model (Local SQLite)

```sql
CREATE TABLE guardian_profile (
  guardian_id   TEXT PRIMARY KEY,
  email         TEXT UNIQUE,
  password_hash TEXT,
  full_name     TEXT,
  role          TEXT CHECK(role IN ('key', 'additional')),
  status        TEXT CHECK(status IN ('active', 'revoked')),
  last_login    TIMESTAMP,
  created_at    TIMESTAMP
);

CREATE TABLE devices (
  device_id     TEXT PRIMARY KEY,
  guardian_id   TEXT,
  fcm_token     TEXT,
  brand         TEXT,
  model         TEXT,
  os_version    TEXT,
  registered_at TIMESTAMP,
  FOREIGN KEY (guardian_id) REFERENCES guardian_profile(guardian_id)
);

CREATE TABLE guardian_children (
  guardian_id  TEXT,
  child_id     TEXT,
  school_id    TEXT,
  relationship TEXT,
  linked_at    TIMESTAMP,
  PRIMARY KEY (guardian_id, child_id),
  FOREIGN KEY (guardian_id) REFERENCES guardian_profile(guardian_id)
);

CREATE TABLE child_access_requests (
  request_id   TEXT PRIMARY KEY,
  requester_id TEXT,
  child_id     TEXT,
  school_id    TEXT,
  relationship TEXT,
  status       TEXT CHECK(status IN ('pending', 'approved', 'denied', 'expired')),
  requested_at TIMESTAMP,
  resolved_at  TIMESTAMP,
  expires_at   TIMESTAMP,
  FOREIGN KEY (requester_id) REFERENCES guardian_profile(guardian_id)
);

CREATE TABLE sync_metadata (
  school_id          TEXT,
  child_id           TEXT,
  last_sync_datetime TIMESTAMP,
  last_sync_status   TEXT,
  sync_error         TEXT,
  PRIMARY KEY (school_id, child_id)
);
```

---

## API Endpoints

| Endpoint                       | Method | Purpose                                           |
| ------------------------------ | ------ | ------------------------------------------------- |
| `/api/auth/register`           | POST   | Register guardian (key or additional)             |
| `/api/auth/login`              | POST   | Validate email + password                         |
| `/api/auth/logout`             | POST   | Clear session + FCM subscriptions                 |
| `/api/auth/forgot-password`    | POST   | Send password reset email                         |
| `/api/auth/reset-password`     | POST   | Submit new password via reset token               |
| `/api/auth/device-register`    | POST   | Register device (device_id, FCM token)            |
| `/api/children/validate`       | POST   | Validate student ID + date of birth               |
| `/api/children/otp/send`       | POST   | Send OTP to key guardian's phone (add-child flow) |
| `/api/children/otp/verify`     | POST   | Verify OTP + link child to key guardian           |
| `/api/children/request-access` | POST   | Additional guardian requests access to a child    |
| `/api/children/access/respond` | POST   | Key guardian approves or denies access request    |
| `/api/children/access/pending` | GET    | Key guardian lists pending access requests        |
| `/api/children/list`           | GET    | List guardian's linked children                   |
| `/api/guardian/revoke`         | POST   | Key guardian revokes additional guardian access   |
| `/api/sync/initial-login`      | POST   | Full data download on first login / new device    |

---

## Session & Security

### JWT Tokens

- Issued on successful login
- Expiry: 30 days (remember me) / 24 hours (default)
- Refresh token valid: 90 days
- On logout: token revoked, FCM subscriptions cleared, local data cleared

### Logout

```
1. Revoke JWT
2. Unsubscribe all FCM topics
3. Clear local SQLite guardian data
4. Redirect to login screen
```

### Password

- Min 8 chars, 1 uppercase, 1 lowercase, 1 number
- Stored as bcrypt hash
- Reset via email link (30-minute expiry, single-use, invalidates all sessions)

### OTP (Add-Child Flow Only)

- 6-digit numeric
- 10-minute expiry
- Rate-limited: 3 attempts per 15-minute window
- All OTP events logged for audit

### Device Binding

- One account = one active device
- New device login warns and locks previous device (requires user confirmation)
- Device ID stored securely; FCM token refreshed automatically
- Remote device lock propagated via sync

### Key Guardian Authority

- Key guardian changes are managed on the school operations side and propagated to the app via sync
- If the current key guardian's authority is removed, their elevated permissions are stripped on next sync

---

## Summary Checklist

- [x] Registration: email + password, same flow for key and additional guardians
- [x] Login: email + password only — no OTP on login
- [x] Key guardian detection: email matching during add-child (registered email vs student master key guardian email)
- [x] OTP: only in add-child flow if detected as key guardian, sent to key guardian's phone
- [x] Key guardian add child: student ID + DOB → email matching → OTP → sync (blocking)
- [x] Additional guardian add child: student ID + DOB → email does not match key guardian email → key guardian in-app approval → sync (blocking)
- [x] New device: warning + previous device lock + fresh sync
- [x] Device registration: device_id, FCM token, brand, model
- [x] Initial sync: blocking with progress bar, retry on failure, offline fallback after 3 retries
- [x] Add more children: same flow per guardian type, sync must complete before next add
- [x] Additional guardian limit: max 3 per child
- [x] Access requests: 48-hour expiry, push notification to key guardian, re-submit on denial (24h wait) or expiry
- [x] Revoke access: key guardian revokes → immediate logout on next sync
- [x] Forgot password: email reset link, 30-minute expiry, single-use, all sessions invalidated
- [x] Key guardian authority changes: managed on school ops side, propagated via sync

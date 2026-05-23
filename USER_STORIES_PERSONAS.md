# User Stories & Personas

> **Related Documents**: [INDEX.md](INDEX.md) | [REQUIREMENTS.md](REQUIREMENTS.md) | [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md) | [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md)  
> **Last Updated**: 2026-05-23  
> **Backlog Items**: ~95 stories across 3 phases  
> **Status**: With full cross-references to requirements, APIs, and databases

---

## How to Use This Document

**Each user story includes explicit cross-references**:

- **Reference**: Shows which [FR-X.Y](#) requirement(s) it implements, which [EP-X](#) API endpoint(s) it uses, and which [TB-X](#) database table(s) it accesses
- **Story Points**: Effort estimation (XS=1-2 hrs, S=1 day, M=2-3 days, L=1-2 weeks, XL=2+ weeks)
- **Dependency**: If it requires another story to be completed first

**Quick Example**: Look at US-001-MVP (Parent Registration):
- ✅ **Requirement**: [FR-1.1](REQUIREMENTS.md#1-authentication--guardian-management)
- ✅ **API**: [EP-1](TECH_ARCHITECTURE.md#ep-1-register-guardian)
- ✅ **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

**For Complete Traceability**: See [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md) for a master map of all stories, requirements, APIs, and tables.

---

## Table of Contents

1. [User Personas](#user-personas)
2. [Backlog Legend & Symbols](#backlog-legend--symbols)
3. [Phase 1 (MVP) Backlog](#phase-1-mvp-backlog) — 35-40 stories, 12 weeks
   - [Epic 1: Authentication & Account Management](#epic-1-authentication--account-management)
   - [Epic 2: Sync & Offline Functionality](#epic-2-sync--offline-functionality)
   - [Epic 3: Dashboard & Profile](#epic-3-dashboard--profile)
   - [Epic 4: School Information & Multi-Tenancy](#epic-4-school-information--multi-tenancy)
   - [Epic 5: Attendance Intelligence](#epic-5-attendance-intelligence)
   - [Epic 6: Homework & Assignments](#epic-6-homework--assignments)
   - [Epic 7: Exams & Results](#epic-7-exams--results)
   - [Epic 8: Daily Class Activity](#epic-8-daily-class-activity)
   - [Epic 9: Class Routine](#epic-9-class-routine)
   - [Epic 10: Notifications](#epic-10-notifications)
   - [Epic 11: Multi-School Support](#epic-11-multi-school-support)
   - [Epic 12: App Performance & Reliability](#epic-12-app-performance--reliability)
   - [Epic 13: Accessibility & Usability](#epic-13-accessibility--usability)
4. [Phase 2 (Enhanced) Backlog](#phase-2-enhanced-backlog) — 30-35 stories, 8 weeks
   - [Behavior & Discipline Analysis](#behavior--discipline-analysis)
   - [Unified Performance Scorecard](#unified-performance-scorecard)
   - [Smart Learning Progress Tracking](#smart-learning-progress-tracking)
   - [Fee Management](#fee-management)
   - [Notice Board](#notice-board)
   - [Event Management](#event-management)
   - [Holiday Calendar](#holiday-calendar)
   - [Issue Messaging System](#issue-messaging-system)
   - [Parent-Teacher Meeting (PTM)](#parent-teacher-meeting-ptm)
   - [Teacher Appointment](#teacher-appointment)
   - [Tomorrow's Due Dashboard](#tomorrows-due-dashboard)
   - [Phone Directory](#phone-directory)
5. [Phase 3 (Health & Growth) Backlog](#phase-3-health--growth-backlog) — 15-20 stories, 8 weeks
   - [Health & Growth Monitoring](#health--growth-monitoring)
   - [Vaccination Record](#vaccination-record)
   - [Daily Parent Tips](#daily-parent-tips)
   - [Classroom Snapshots](#classroom-snapshots)
   - [Advanced Calendar Integration](#advanced-calendar-integration)
6. [Backlog Summary](#backlog-summary)

---

## User Personas

Every user of the Guardian Parent App is one of exactly two types. There is no generic "Guardian" type — all accounts are either Key Guardian or Additional Guardian.

---

### Persona 1: Key Guardian (Primary Parent)

**Name**: Ahmed Hassan  
**Age**: 38  
**Location**: Dhaka, Bangladesh  
**Occupation**: School teacher (authority figure)  
**Tech Savvy**: High (uses apps, understands tech)  

**Role in System**: Email matches `student_master.key_guardian_email` set by the school. Designated primary contact. Full access to all features.

**Goals**:
- Be the primary contact for my child's school
- Approve or deny other guardians' access to my child's info
- Have full control and visibility over all data
- Manage who has access to my child's information
- Comprehensive view of child's academic and behavioral development

**Pain Points**:
- Need to verify other guardians are legitimate
- Worried about data privacy
- Need control over who sees child's info
- Want detailed insights for better decision-making

**Behavior**:
- Checks app daily (morning 8-9 AM and evening 8-10 PM)
- Uses phone exclusively (no desktop)
- Rarely offline (maybe 1-2 times/month)
- Shares responsibility with spouse but retains primary control

**App Usage**:
- Opens app 5-7 times per week (daily habit)
- Session length: 10-15 minutes
- Primary actions: View all data, approve/revoke additional guardians, check alerts
- **Receives**: All push notifications (homework, exams, attendance, behavior, announcements)

---

### Persona 2: Additional Guardian (Secondary Parent/Relative)

**Name**: Aisha Khan (Aunt)  
**Age**: 32  
**Location**: Dhaka, Bangladesh  
**Occupation**: Homemaker, helps with childcare  
**Tech Savvy**: Low-Medium (uses WhatsApp, needs simple UI)  

**Role in System**: Email does NOT match key guardian email. Must request access; Key Guardian must approve. Read-only access to child data. No push notifications.

**Goals**:
- Help monitor my niece/nephew's school activities
- Know about homework and exams
- Support the primary guardian
- Access child's data when needed (without being notified constantly)

**Pain Points**:
- Depends on Key Guardian for access approval
- Limited visibility into what child is doing
- Gets confused by complicated apps
- Finds out about updates only by opening the app (no push notifications)

**Behavior**:
- Checks app when helping with homework (evenings)
- Limited internet (mostly WiFi at home)
- Often goes offline (travel, home without WiFi)
- Uncomfortable with technology, needs simple interface

**App Usage**:
- Opens app 2-3 times per week (only when needed)
- Session length: 2-5 minutes
- Primary actions: Check homework, view class activities, see attendance
- **Receives**: No push notifications — sees latest data when opening the app after sync

---

## Backlog Legend & Symbols

### **Story ID Format**
```
US-001-MVP  = User Story 001 in MVP Phase
US-042-P2   = User Story 042 in Phase 2
US-089-P3   = User Story 089 in Phase 3
```

### **Priority Levels**
- 🔴 **MUST** = Critical for MVP launch (blocks release)
- 🟡 **SHOULD** = Important, can defer 1-2 sprints
- 🟢 **NICE** = Nice-to-have, Phase 2+

### **Story Points** (Fibonacci Scale)
- **XS** (1-2): Simple form, basic validation
- **S** (3-5): Standard CRUD, one integration
- **M** (8-13): Complex logic, multiple integrations
- **L** (20-40): Very complex, major refactor needed
- **XL** (50+): Epic-level, split into smaller stories

### **Dependencies**
- Shows what story must complete first
- Critical path identified for MVP

### **Status Options**
- ⬜ Not Started (default)
- 🔵 In Progress
- ✅ Completed

---

## Phase 1 (MVP) Backlog

**Duration**: Weeks 1-12  
**Target**: 5,000 initial users  
**Goal**: Launch-ready core features  
**Stories**: 35-40  
**Total Story Points**: 150-180

### Epic 1: Authentication & Account Management

#### **US-001-MVP: Parent Registration**
- **Priority**: 🔴 MUST
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent, I want to register with email and password so that I can create an account
- **Acceptance Criteria**:
  - Email format validated
  - Password meets requirements (8+ chars, 1 uppercase, 1 lowercase, 1 digit)
  - Email uniqueness checked
  - Account created successfully
  - Confirmation notification sent
- **Dependencies**: None
- **Reference**: [REQUIREMENTS.md#authentication](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Register](TECH_ARCHITECTURE.md#register-endpoint)

---

#### **US-002-MVP: Guardian Login**
- **Priority**: 🔴 MUST
- **Points**: S (3)
- **Persona**: All
- **Story**: As a registered parent, I want to login with email and password so that I can access my child's information
- **Acceptance Criteria**:
  - Email and password validated
  - JWT token issued on success
  - Error shown on wrong credentials
  - Account locked after 3 failed attempts (15 min lockout)
  - No OTP required on login
- **Dependencies**: US-001-MVP
- **Reference**: [REQUIREMENTS.md#guardian-login](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Login](TECH_ARCHITECTURE.md#login-endpoint)

---

#### **US-003-MVP: Key Guardian Add First Child (With OTP)**
- **Priority**: 🔴 MUST
- **Points**: M (13)
- **Persona**: Key Guardian, Guardian
- **Story**: As a key guardian, I want to add my child using student ID and DOB and verify with OTP so that I can access my child's complete school information
- **Acceptance Criteria**:
  - Student ID and DOB validated against school master
  - Email matches key guardian email (auto-detection)
  - OTP generated and sent to registered phone
  - OTP masked on display (last 2 digits visible)
  - Maximum 3 wrong attempts allowed
  - Resend OTP available after 30 seconds
  - Child linked after successful OTP verification
  - Initial sync triggered (blocking)
- **Dependencies**: US-002-MVP
- **Reference**: [REQUIREMENTS.md#otp-verification](REQUIREMENTS.md#1-authentication--guardian-management) | [API: OTP](TECH_ARCHITECTURE.md#otp-endpoints)

---

#### **US-004-MVP: Additional Guardian Request Access**
- **Priority**: 🔴 MUST
- **Points**: M (8)
- **Persona**: Guardian (Additional)
- **Story**: As a parent (not the key guardian), I want to request access to my child's profile so that the key guardian can approve my access
- **Acceptance Criteria**:
  - Student ID and DOB validated
  - Email does NOT match key guardian email
  - Request created with 48-hour expiry
  - Key guardian receives push notification
  - Additional guardian gets confirmation (awaiting approval)
  - Can re-submit after 24 hours if denied
  - Request expires automatically after 48 hours
- **Dependencies**: US-002-MVP
- **Reference**: [REQUIREMENTS.md#additional-guardian-add-child](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Request Access](TECH_ARCHITECTURE.md#access-request-endpoint)

---

#### **US-005-MVP: Key Guardian Approve/Deny Access Request**
- **Priority**: 🔴 MUST
- **Points**: M (8)
- **Persona**: Key Guardian
- **Story**: As a key guardian, I want to approve or deny additional guardian requests so that I can control who has access to my child's information
- **Acceptance Criteria**:
  - List of pending requests shown with requester details
  - Approve button links additional guardian and triggers sync
  - Deny button blocks access and allows re-submission after 24h
  - Key Guardian receives push notification of new request; Additional Guardian sees decision status in-app on next sync (no push notification to Additional Guardian)
  - Child linked immediately on approval
- **Dependencies**: US-004-MVP
- **Reference**: [REQUIREMENTS.md#1-authentication--guardian-management](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Respond](TECH_ARCHITECTURE.md#approval-endpoint)

---

#### **US-006-MVP: New Device Login Warning**
- **Priority**: 🔴 MUST
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent logging in from a new device, I want to see a warning that my previous device will be locked so that I understand the security implications
- **Acceptance Criteria**:
  - Warning dialog shown on new device detection
  - Old device_id will be locked (explained)
  - Continue/Cancel buttons
  - On continue: Old device marked as locked, new device registered
  - Old device logged out on next sync
- **Dependencies**: US-002-MVP
- **Reference**: [REQUIREMENTS.md#new-device-login](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Login](TECH_ARCHITECTURE.md#login-endpoint)

---

#### **US-007-MVP: Revoke Additional Guardian Access**
- **Priority**: 🔴 MUST
- **Points**: S (5)
- **Persona**: Key Guardian
- **Story**: As a key guardian, I want to revoke an additional guardian's access so that I can remove them from viewing my child's information
- **Acceptance Criteria**:
  - List of additional guardians shown
  - Revoke button with confirmation dialog
  - Revoked Additional Guardian is force-logged out on next delta sync (within 10 minutes)
  - All linked children access removed
  - Revoked Additional Guardian sees in-app message on next sync: "Your access has been revoked. Contact the key guardian." (no push notification sent)
- **Dependencies**: US-005-MVP
- **Reference**: [REQUIREMENTS.md#1-authentication--guardian-management](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Revoke](TECH_ARCHITECTURE.md#revoke-endpoint)

---

#### **US-008-MVP: Forgot Password**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent who forgot my password, I want to reset it via email so that I can regain access to my account
- **Acceptance Criteria**:
  - Email field on forgot password screen
  - Reset link sent to email (valid 30 minutes)
  - Link is single-use
  - New password entered and confirmed
  - All active sessions invalidated
  - Confirmation message shown
  - Must login again on all devices
- **Dependencies**: US-002-MVP
- **Reference**: [REQUIREMENTS.md#forgot-password](REQUIREMENTS.md#1-authentication--guardian-management) | [API: Reset](TECH_ARCHITECTURE.md#password-reset-endpoint)

---

### Epic 2: Sync & Offline Functionality

#### **US-009-MVP: Initial Full Sync on First Login**
- **Priority**: 🔴 MUST
- **Points**: L (20)
- **Persona**: All
- **Story**: As a parent after adding my first child, I want the app to download all my child's data so that I can view information even without internet
- **Acceptance Criteria**:
  - Blocking sync with progress bar (3 steps: registering device, downloading school data, downloading student data)
  - All academic year and continuous data downloaded
  - SQLite database created locally
  - School branding applied
  - FCM topics subscribed
  - "Setup complete ✓" message shown
  - Timeout after 3 failures; fallback to offline mode
  - Can retry sync
- **Dependencies**: US-003-MVP or US-005-MVP
- **Reference**: [REQUIREMENTS.md#initial-sync](REQUIREMENTS.md#2-sync--offline-functionality) | [API: Initial Sync](TECH_ARCHITECTURE.md#initial-sync-endpoint)

---

#### **US-010-MVP: Background 10-Minute Delta Sync**
- **Priority**: 🔴 MUST
- **Points**: L (20)
- **Persona**: All
- **Story**: As a parent with internet connection, I want the app to silently sync new data every 10 minutes so that I always see the latest information
- **Acceptance Criteria**:
  - Sync runs every 10 minutes (configurable)
  - Only changed data fetched (delta sync)
  - Local SQLite merged with new data
  - "Last synced: just now" indicator shown
  - No interruption to user
  - Works across multiple schools (per-school sync requests)
  - Retry on failure at next cycle
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#background-10-min-sync](REQUIREMENTS.md#2-sync--offline-functionality) | [API: Delta Sync](TECH_ARCHITECTURE.md#delta-sync-endpoint)

---

#### **US-011-MVP: Offline Data Access**
- **Priority**: 🔴 MUST
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent without internet connection, I want to view my child's information from cached local data so that I stay informed even when offline
- **Acceptance Criteria**:
  - All read features work offline (attendance, homework, exams, etc.)
  - "Offline" indicator shown clearly
  - Data staleness clearly displayed:
    - < 24h: "Last updated: 2 hours ago"
    - 24-48h: Warning banner shown
    - > 7 days: Alert shown
  - No edits allowed (read-only mode)
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#offline-experience](REQUIREMENTS.md#2-sync--offline-functionality)

---

#### **US-012-MVP: Automatic Sync on Connection Return**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent who regained internet connection, I want the app to automatically sync new data so that I see the latest information without manual refresh
- **Acceptance Criteria**:
  - Network state monitored
  - Sync triggered immediately when connection detected
  - No manual "Sync" button needed
  - UI updated with fresh data
  - "Last synced: just now" shown
- **Dependencies**: US-010-MVP
- **Reference**: [REQUIREMENTS.md#automatic-sync](REQUIREMENTS.md#2-sync--offline-functionality)

---

#### **US-013-MVP: Revocation Detection via Sync**
- **Priority**: 🔴 MUST
- **Points**: M (8)
- **Persona**: Additional Guardian
- **Story**: As a revoked additional guardian, I want to be informed when my access is revoked so that I understand I can no longer access the app
- **Acceptance Criteria**:
  - Delta sync response includes revocation_status flag
  - Local data cleared immediately on detection
  - Force logout triggered
  - In-app message shown: "Your access has been revoked. Contact the key guardian."
  - Redirect to login screen
  - No push notification sent (Additional Guardian receives no push notifications)
  - Detection within 10 minutes via next scheduled delta sync
- **Dependencies**: US-010-MVP, US-007-MVP
- **Reference**: [REQUIREMENTS.md#2-sync--offline-functionality](REQUIREMENTS.md#2-sync--offline-functionality) | [API: Delta Sync](TECH_ARCHITECTURE.md#ep-7-delta-sync)

---

### Epic 3: Dashboard & Profile

#### **US-014-MVP: View Child Dashboard**
- **Priority**: 🔴 MUST
- **Points**: M (13)
- **Persona**: All
- **Story**: As a parent, I want to see my child's academic overview on the home screen so that I quickly understand my child's current performance
- **Acceptance Criteria**:
  - Student name, photo, class, section displayed
  - School logo and branding applied
  - Quick stats shown:
    - Attendance % (last 30 days)
    - Next homework due (with subject)
    - Upcoming exam (date, subject)
    - Alert badge if low attendance (< 75%)
  - Last synced timestamp shown
  - Loads instantly from local cache
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#dashboard-display](REQUIREMENTS.md#3-dashboard--student-profile) | [API: Dashboard](TECH_ARCHITECTURE.md#dashboard-endpoint)

---

#### **US-015-MVP: Multi-Child Profile Switching**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: Guardian (with multiple children)
- **Story**: As a parent with multiple children, I want to quickly switch between children so that I can view information for each child
- **Acceptance Criteria**:
  - Dropdown or carousel shows all linked children
  - Tap child loads their dashboard instantly
  - Last selected child remembered
  - All data reloads for selected child
  - Switch happens in < 500ms
- **Dependencies**: US-014-MVP
- **Reference**: [REQUIREMENTS.md#multi-child-switching](REQUIREMENTS.md#3-dashboard--student-profile)

---

#### **US-016-MVP: View Detailed Student Profile**
- **Priority**: 🟡 SHOULD
- **Points**: S (3)
- **Persona**: All
- **Story**: As a parent, I want to see all of my child's profile information so that I have complete details about my child's school enrollment
- **Acceptance Criteria**:
  - Student name, photo, DOB, class, section displayed
  - Curriculum and session shown
  - Address and guardian information shown
  - Emergency contacts displayed
  - Profile is read-only
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#student-profile-display](REQUIREMENTS.md#3-dashboard--student-profile)

---

### Epic 4: School Information & Multi-Tenancy

#### **US-017-MVP: School Selection/Discovery**
- **Priority**: 🔴 MUST
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent, I want to discover my child's school in the app so that I can select the correct school during registration
- **Acceptance Criteria**:
  - Search field with school name
  - Matching schools listed (name, location, code)
  - Select school to proceed
  - School details (logo, contact) displayed
- **Dependencies**: US-001-MVP
- **Reference**: [REQUIREMENTS.md#school-discovery](REQUIREMENTS.md#4-school-information--multi-tenancy) | [API: School Search](TECH_ARCHITECTURE.md#school-endpoints)

---

#### **US-018-MVP: School Branding in App**
- **Priority**: 🟡 SHOULD
- **Points**: S (3)
- **Persona**: All
- **Story**: As a parent, I want to see my school's branding throughout the app so that I feel connected to the school
- **Acceptance Criteria**:
  - School logo displayed on header
  - Primary and secondary colors applied to UI
  - School name and slogan visible
  - Branding consistent across all screens
  - Applied from initial sync
- **Dependencies**: US-017-MVP
- **Reference**: [REQUIREMENTS.md#school-branding](REQUIREMENTS.md#4-school-information--multi-tenancy)

---

#### **US-019-MVP: View School Contacts**
- **Priority**: 🟡 SHOULD
- **Points**: S (3)
- **Persona**: All
- **Story**: As a parent, I want to quickly access school contact information so that I can reach out to school staff when needed
- **Acceptance Criteria**:
  - School contact list displayed (principal, admin, class teacher)
  - Phone numbers and emails shown
  - Tap phone → dial app opens
  - Tap email → email app opens
  - Directory synced with each update
- **Dependencies**: US-017-MVP
- **Reference**: [REQUIREMENTS.md#contact-directory](REQUIREMENTS.md#4-school-information--multi-tenancy) | [API: Directory](TECH_ARCHITECTURE.md#contact-endpoints)

---

### Epic 5: Attendance Intelligence

#### **US-020-MVP: View Daily Attendance Calendar**
- **Priority**: 🔴 MUST
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent, I want to see my child's daily attendance status in a calendar so that I know if my child attended school
- **Acceptance Criteria**:
  - Calendar view of last 30 days
  - Color coding:
    - Green = Present
    - Red = Absent
    - Yellow = Leave
    - Orange = Half-day
  - Tap date shows remarks (if any)
  - Monthly attendance % calculated and displayed
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#daily-attendance](REQUIREMENTS.md#5-attendance-intelligence) | [API: Attendance](TECH_ARCHITECTURE.md#attendance-endpoints)

---

#### **US-021-MVP: Low Attendance Alert**
- **Priority**: 🔴 MUST
- **Points**: S (5)
- **Persona**: Key Guardian only
- **Story**: As a Key Guardian, I want to be notified when my child's attendance drops below 75% so that I can take action to improve attendance
- **Acceptance Criteria**:
  - Threshold: < 75% (configurable by school)
  - Push notification sent to Key Guardian only
  - In-app notification badge shown
  - Message: "[Child Name]'s attendance dropped to X%"
  - One alert per day (if condition persists)
  - Additional Guardian sees the attendance data in-app but receives no push alert
- **Dependencies**: US-020-MVP, US-010-MVP
- **Reference**: [REQUIREMENTS.md#5-attendance-intelligence](REQUIREMENTS.md#5-attendance-intelligence) | [API: Attendance](TECH_ARCHITECTURE.md#attendance-endpoints)

---

#### **US-022-MVP: Attendance Trend Analysis**
- **Priority**: 🟡 SHOULD
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent, I want to see my child's attendance trend over time so that I understand attendance patterns
- **Acceptance Criteria**:
  - Weekly trend chart (last 4 weeks)
  - Monthly trend chart (last 6 months)
  - Comparison with previous month shown
  - Visual indicators for trends (up/down)
- **Dependencies**: US-020-MVP
- **Reference**: [REQUIREMENTS.md#attendance-analytics](REQUIREMENTS.md#5-attendance-intelligence)

---

### Epic 6: Homework & Assignments

#### **US-023-MVP: View Homework List**
- **Priority**: 🔴 MUST
- **Points**: M (13)
- **Persona**: All
- **Story**: As a parent, I want to see my child's homework with due dates so that I can track assignment deadlines
- **Acceptance Criteria**:
  - Homework list showing:
    - Subject
    - Title and description
    - Due date
    - Status (assigned, submitted, evaluated, late, overdue)
    - Teacher marks and feedback (if evaluated)
  - Filter by subject
  - Sort by due date (default)
  - Mark as viewed
  - Teacher feedback displayed clearly
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#homework-display](REQUIREMENTS.md#8-homework--assignment-tracker) | [API: Homework](TECH_ARCHITECTURE.md#homework-endpoints)

---

#### **US-024-MVP: Homework Due Reminder**
- **Priority**: 🟡 SHOULD
- **Points**: S (3)
- **Persona**: Key Guardian only
- **Story**: As a Key Guardian, I want to receive a push reminder 1 day before homework is due so that I can ensure my child submits on time
- **Acceptance Criteria**:
  - Push notification sent to Key Guardian 24 hours before due date
  - "Due tomorrow" label shown in app for both personas
  - Notification includes subject and title
  - Delivery success > 99%
  - Additional Guardian sees "Due tomorrow" label in-app only (no push notification)
- **Dependencies**: US-023-MVP, FCM setup
- **Reference**: [REQUIREMENTS.md#8-homework--assignment-tracker](REQUIREMENTS.md#8-homework--assignment-tracker)

---

### Epic 7: Exams & Results

#### **US-025-MVP: View Exam Schedule**
- **Priority**: 🔴 MUST
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent, I want to see upcoming exams with dates and subjects so that I know when my child has exams
- **Acceptance Criteria**:
  - Exam list showing:
    - Exam type (test, monthly, term, annual)
    - Date and time
    - Subject
    - Duration
  - Upcoming exams highlighted
  - Chronological order
  - 1-day before reminder
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#exam-schedule](REQUIREMENTS.md#9-examination--result-management) | [API: Exams](TECH_ARCHITECTURE.md#exam-endpoints)

---

#### **US-026-MVP: View Exam Results**
- **Priority**: 🔴 MUST
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent, I want to see my child's exam results with marks and grades so that I understand my child's academic performance
- **Acceptance Criteria**:
  - Results show:
    - Subject
    - Marks obtained / total marks
    - Grade (A/B/C/D/F)
    - Percentile
  - Subject-wise breakdown
  - Published on exam date + 1-2 days
  - Past results available in history
- **Dependencies**: US-025-MVP
- **Reference**: [REQUIREMENTS.md#exam-results](REQUIREMENTS.md#9-examination--result-management) | [API: Results](TECH_ARCHITECTURE.md#exam-endpoints)

---

#### **US-027-MVP: Exam Performance Analytics**
- **Priority**: 🟡 SHOULD
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent, I want to see comparative analytics like class average and percentile so that I understand how my child performs relative to peers
- **Acceptance Criteria**:
  - Class average shown (anonymized)
  - Student percentile calculated
  - Performance band shown (Advanced/Proficient/Developing)
  - Subject-wise comparison
  - Trend line for subject over exams
- **Dependencies**: US-026-MVP
- **Reference**: [REQUIREMENTS.md#comparative-analytics](REQUIREMENTS.md#9-examination--result-management)

---

### Epic 8: Daily Class Activity

#### **US-028-MVP: View Daily Class Updates**
- **Priority**: 🟡 SHOULD
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent, I want to see what was taught in class each day so that I understand my child's learning progress
- **Acceptance Criteria**:
  - Daily activity feed (last 7 days)
  - Shows:
    - Date and subject
    - Topics taught
    - Chapters covered
    - Classroom activities
    - Teacher remarks
  - Load older functionality
  - Filter by subject
  - Card view (recent first)
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#daily-activity-display](REQUIREMENTS.md#7-daily-class-activity) | [API: Activity](TECH_ARCHITECTURE.md#activity-endpoints)

---

### Epic 9: Class Routine

#### **US-029-MVP: View Class Timetable**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent, I want to see my child's weekly class routine so that I know what subjects are scheduled
- **Acceptance Criteria**:
  - Weekly timetable displayed
  - Each slot shows:
    - Subject name
    - Teacher name
    - Time
    - Location (if available)
  - Today's schedule highlighted
  - Tap teacher shows contact details
- **Dependencies**: US-009-MVP
- **Reference**: [REQUIREMENTS.md#class-routine-display](REQUIREMENTS.md#6-class-routine) | [API: Routine](TECH_ARCHITECTURE.md#routine-endpoints)

---

### Epic 10: Notifications

#### **US-030-MVP: Push Notifications for Key Events**
- **Priority**: 🔴 MUST
- **Points**: M (13)
- **Persona**: Key Guardian only
- **Story**: As a Key Guardian, I want to receive push notifications for important events so that I stay informed without checking the app manually
- **Acceptance Criteria**:
  - Push notifications sent to Key Guardian only for:
    - Homework due tomorrow
    - Exam announced / scheduled
    - Low attendance alert
    - Behavioral concerns
    - Assignment evaluated
  - Additional Guardian receives no push notifications
  - Tap notification opens relevant section
  - Notification preference settings available (Key Guardian only)
  - Do Not Disturb hours respected (e.g., 9 PM - 8 AM)
  - FCM used for delivery to Key Guardian devices only
- **Dependencies**: FCM setup
- **Reference**: [REQUIREMENTS.md#10-notifications-core-implementation](REQUIREMENTS.md#10-notifications-core-implementation) | [API: Notifications](TECH_ARCHITECTURE.md#notification-endpoints)

---

#### **US-031-MVP: Notification Preferences**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: Key Guardian only
- **Story**: As a Key Guardian, I want to customize my notification settings so that I only receive notifications I care about
- **Acceptance Criteria**:
  - Enable/disable per category (homework, exams, attendance, etc.)
  - Set quiet hours (no notifications during sleep)
  - Frequency control (e.g., once per day for repeat alerts)
  - Settings synced across Key Guardian devices only
  - Not applicable to Additional Guardian (receives no push notifications)
- **Dependencies**: US-030-MVP
- **Reference**: [REQUIREMENTS.md#10-notifications-core-implementation](REQUIREMENTS.md#10-notifications-core-implementation)

---

#### **US-032-MVP: Notification History**
- **Priority**: 🟢 NICE
- **Points**: S (3)
- **Persona**: Key Guardian only
- **Story**: As a Key Guardian, I want to view past notifications so that I can review messages I might have missed
- **Acceptance Criteria**:
  - Notification center / history view (Key Guardian only)
  - Notifications sortable by date
  - Mark as read/unread
  - Search notifications
  - Clear all option
  - Not shown to Additional Guardian (no notification history)
- **Dependencies**: US-030-MVP
- **Reference**: [REQUIREMENTS.md#10-notifications-core-implementation](REQUIREMENTS.md#10-notifications-core-implementation)

---

### Epic 11: Multi-School Support

#### **US-033-MVP: Add Child from Different School**
- **Priority**: 🟡 SHOULD
- **Points**: M (8)
- **Persona**: Guardian (multiple schools)
- **Story**: As a parent with children in different schools, I want to add another child from a different school so that I can manage all children in one app
- **Acceptance Criteria**:
  - Same registration flow as first child
  - Different school_id detected
  - Separate sync cycle per school
  - Quick stats per school shown
- **Dependencies**: US-003-MVP or US-005-MVP
- **Reference**: [REQUIREMENTS.md#multi-school-support](REQUIREMENTS.md#4-school-information--multi-tenancy)

---

#### **US-034-MVP: View School Context Properly**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: Guardian (multiple schools)
- **Story**: As a parent with children in multiple schools, I want to see which school each piece of information belongs to so that I don't get confused
- **Acceptance Criteria**:
  - School name displayed in context
  - School branding shown appropriately
  - Data filtered per school
  - Last synced time per school shown
- **Dependencies**: US-033-MVP
- **Reference**: [REQUIREMENTS.md#multi-tenancy-isolation](REQUIREMENTS.md#4-school-information--multi-tenancy)

---

### Epic 12: App Performance & Reliability

#### **US-035-MVP: App Loads Quickly**
- **Priority**: 🟡 SHOULD
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent, I want the app to load quickly so that I don't get frustrated waiting
- **Acceptance Criteria**:
  - App startup: < 3 seconds
  - Dashboard load: < 1 second
  - Page transition: < 500 ms
  - Smooth performance on 2GB RAM devices
  - No crashes or hangs
- **Dependencies**: All
- **Reference**: [REQUIREMENTS.md#performance-requirements](REQUIREMENTS.md#system-wide-non-functional-requirements)

---

#### **US-036-MVP: Graceful Error Handling**
- **Priority**: 🟡 SHOULD
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent, I want meaningful error messages when something goes wrong so that I understand what happened and how to fix it
- **Acceptance Criteria**:
  - User-friendly error messages (not technical)
  - Retry buttons provided
  - Offline mode graceful fallback
  - Sync failures don't crash app
  - Clear status indicators
- **Dependencies**: All
- **Reference**: [REQUIREMENTS.md#error-handling](REQUIREMENTS.md#2-sync--offline-functionality)

---

### Epic 13: Accessibility & Usability

#### **US-037-MVP: Bangla Language Support**
- **Priority**: 🔴 MUST
- **Points**: M (13)
- **Persona**: All
- **Story**: As a parent, I want to use the app in Bangla so that I understand all content clearly
- **Acceptance Criteria**:
  - All UI text in Bangla (primary language)
  - English also available (secondary)
  - Language toggle in settings
  - Date format: DD/MM/YYYY
  - Currency: ৳ (BDT)
  - Phone format: +880 with validation
  - Numbers in Bangla numerals (optional)
- **Dependencies**: All
- **Reference**: [REQUIREMENTS.md#localization](REQUIREMENTS.md#system-wide-non-functional-requirements)

---

#### **US-038-MVP: Mobile-Responsive Design**
- **Priority**: 🔴 MUST
- **Points**: L (20)
- **Persona**: All
- **Story**: As a parent using a mobile phone, I want the app to be optimized for small screens so that I can easily tap buttons and read text
- **Acceptance Criteria**:
  - Works on 4.5" - 6.5" phones
  - Text readable without zooming
  - Buttons easily tappable (48x48 dp minimum)
  - No horizontal scrolling
  - Landscape mode supported
  - Touch-friendly UI
- **Dependencies**: All
- **Reference**: [REQUIREMENTS.md#device-support](REQUIREMENTS.md#system-wide-non-functional-requirements)

---

#### **US-039-MVP: Low-Bandwidth Optimization**
- **Priority**: 🟡 SHOULD
- **Points**: M (8)
- **Persona**: All
- **Story**: As a parent on slow 3G connection, I want the app to work efficiently so that I don't waste expensive data
- **Acceptance Criteria**:
  - Images optimized and compressed
  - Lazy loading for lists
  - Delta sync minimizes bandwidth
  - Works on 3G (though slower)
  - Data usage tracked (optional)
  - Monthly usage < 100 MB reasonable
- **Dependencies**: All
- **Reference**: [REQUIREMENTS.md#bandwidth-optimization](REQUIREMENTS.md#2-sync--offline-functionality)

---

#### **US-040-MVP: Battery Efficient**
- **Priority**: 🟢 NICE
- **Points**: S (5)
- **Persona**: All
- **Story**: As a parent, I want the app to not drain my phone battery so that I can use it throughout the day
- **Acceptance Criteria**:
  - Background sync uses < 3% battery per 10-minute cycle
  - No excessive network requests
  - Location not tracked
  - 10-minute sync interval (not more frequent)
  - Efficient database queries
- **Dependencies**: All
- **Reference**: [REQUIREMENTS.md#battery-efficiency](REQUIREMENTS.md#2-sync--offline-functionality)

---

## Phase 2 (Enhanced) Backlog

**Duration**: Weeks 13-20  
**Stories**: 30-35  
**Total Story Points**: 120-150  

**[See Feature Specifications →](REQUIREMENTS.md#phase-2-enhanced-engagement-features)**

---

### Behavior & Discipline Analysis

**Reference**: [FR-11.x](REQUIREMENTS.md#11-behavior--discipline-analysis)

- **US-041-P2**: View Behavior Scores & Trend
- **US-042-P2**: Low Behavior Score Alert *(Key Guardian push notification only)*

---

### Unified Performance Scorecard

**Reference**: [FR-12.x](REQUIREMENTS.md#12-unified-performance-scorecard)

- **US-043-P2**: View Performance Scorecard

---

### Smart Learning Progress Tracking

**Reference**: [FR-13.x](REQUIREMENTS.md#13-smart-learning-progress-tracking)

- **US-044-P2**: View Chapter-wise Progress

---

### Fee Management

**Reference**: [FR-14.x](REQUIREMENTS.md#14-fee-management)

- **US-045-P2**: View Fee Due Notices
- **US-046-P2**: Make Fee Payment (bKash)
- **US-047-P2**: Fee Payment Confirmation *(Key Guardian push notification only)*

---

### Notice Board

**Reference**: [FR-15.x](REQUIREMENTS.md#15-notice-board)

- **US-048-P2**: View School Notices
- **US-049-P2**: New Notice Alert *(Key Guardian push notification only)*

---

### Event Management

**Reference**: [FR-16.x](REQUIREMENTS.md#16-event-management)

- **US-050-P2**: View School Events
- **US-051-P2**: Event Reminder *(Key Guardian push notification only)*

---

### Holiday Calendar

**Reference**: [FR-17.x](REQUIREMENTS.md#17-holiday-calendar)

- **US-052-P2**: View Holiday Calendar

---

### Issue Messaging System

**Reference**: [FR-18.x](REQUIREMENTS.md#18-issue-messaging-system)

- **US-053-P2**: Submit Issue to School
- **US-054-P2**: Issue Status Update *(Key Guardian push notification only)*

---

### Parent-Teacher Meeting (PTM)

**Reference**: [FR-19.x](REQUIREMENTS.md#19-parent-teacher-meeting-ptm)

- **US-055-P2**: View PTM Schedule
- **US-056-P2**: PTM Reminder *(Key Guardian push notification only)*

---

### Teacher Appointment

**Reference**: [FR-20.x](REQUIREMENTS.md#20-teacher-appointment)

- **US-057-P2**: Request Teacher Appointment
- **US-058-P2**: Appointment Confirmation *(Key Guardian push notification only)*

---

### Tomorrow's Due Dashboard

**Reference**: [FR-21.x](REQUIREMENTS.md#21-tomorrows-due-dashboard)

- **US-059-P2**: View Tomorrow's Homework & Exams

---

### Phone Directory

**Reference**: [FR-22.x](REQUIREMENTS.md#22-phone-directory)

- **US-060-P2**: View & Call School Staff Directory

---

## Phase 3 (Health & Growth) Backlog

**Duration**: Weeks 21-28  
**Stories**: 15-20  
**Total Story Points**: 80-100  

**[See Feature Specifications →](REQUIREMENTS.md#phase-3-health--growth-wellness-features)**

---

### Health & Growth Monitoring

**Reference**: [FR-23.x](REQUIREMENTS.md#23-health--growth-monitoring)

- **US-062-P3**: View Height & Weight Records
- **US-063-P3**: View Growth Charts

---

### Vaccination Record

**Reference**: [FR-24.x](REQUIREMENTS.md#24-vaccination-record)

- **US-064-P3**: View Vaccination History
- **US-065-P3**: Vaccination Due Reminder *(Key Guardian push notification only)*

---

### Daily Parent Tips

**Reference**: [FR-25.x](REQUIREMENTS.md#25-daily-parent-tips)

- **US-066-P3**: Receive Daily Parenting Tips *(Key Guardian push notification only)*

---

### Classroom Snapshots

**Reference**: [FR-26.x](REQUIREMENTS.md#26-classroom-snapshots)

- **US-067-P3**: View Classroom Snapshots

---

### Advanced Calendar Integration

**Reference**: [FR-27.x](REQUIREMENTS.md#27-advanced-calendar-integration)

- **US-068-P3**: Sync School Events to Device Calendar

---

## Backlog Summary

### **By Priority**

| Priority | MVP | Phase 2 | Phase 3 | Total | % of Total |
|----------|-----|---------|---------|-------|------------|
| 🔴 MUST | 22 | 8 | 2 | **32** | 36% |
| 🟡 SHOULD | 15 | 18 | 10 | **43** | 48% |
| 🟢 NICE | 3 | 9 | 8 | **20** | 22% |
| **TOTAL** | **40** | **35** | **20** | **95** | 100% |

### **By Story Points**

| Points | Count | Total Hours (@ 7.5h/point) | Effort |
|--------|-------|---------------------------|--------|
| XS (1-2) | 8 | 12-16 hrs | Low |
| S (3-5) | 28 | 84-140 hrs | Low-Med |
| M (8-13) | 38 | 285-570 hrs | Medium |
| L (20-40) | 15 | 300-450 hrs | High |
| XL (50+) | 6 | 450+ hrs | Very High |
| **TOTAL** | **95** | **~2,500 hours** | **~15-16 weeks** |

### **By Phase**

| Phase | Duration | Stories | Avg Points | Total Points | Velocity (pts/week) |
|-------|----------|---------|------------|--------------|-------------------|
| MVP | 12 weeks | 40 | 4.5 | 180 | 15 pts/week |
| Phase 2 | 8 weeks | 35 | 4.0 | 140 | 17.5 pts/week |
| Phase 3 | 8 weeks | 20 | 4.0 | 80 | 10 pts/week |

### **Critical Path (MVP Dependencies)**

```
1. US-001 (Register)
   ↓
2. US-002 (Login)
   ↓
3. US-003/004 (Add Child)
   ↓
4. US-009/010 (Initial & Delta Sync) ← CRITICAL
   ↓
5. US-014 (Dashboard) ← Can parallel US-020, 023, 025, 028, 029
   ↓
6. US-030 (Notifications) ← Depends on sync
   ↓
7. US-037/038 (Bangla + Mobile UI) ← Can parallel all
```

---

## How to Use This Backlog

### **For Sprint Planning**
1. Pick stories from top of MVP backlog (prioritized)
2. Check dependencies before assigning
3. Mix XS/S stories with M stories
4. Avoid assigning XL stories without splitting
5. Aim for 120-150 points per 2-week sprint

### **For Estimation**
- Velocity tracking: Track completed points per sprint
- Burndown: Plot remaining points over sprints
- Forecast: (Remaining points) / (avg velocity) = weeks remaining

### **For Status Tracking**
- Update story status as: ⬜ Not Started → 🔵 In Progress → ✅ Completed
- Track blockers and dependencies
- Report on must-have vs nice-to-have completion

---

## Story Point Guidance

### **How Many Points = How Long?**

Assuming 7.5 hours productive work per day:

- **XS (1-2 pts)** = 1-2 hours = half morning
- **S (3-5 pts)** = 3-5 hours = full day
- **M (8-13 pts)** = 8-13 hours = 2-3 days
- **L (20-40 pts)** = 20-40 hours = 1-2 weeks
- **XL (50+ pts)** = Needs splitting

---

## References

- [REQUIREMENTS.md](REQUIREMENTS.md) - Feature details and acceptance criteria
- [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md) - API contracts and database schemas
- [INDEX.md](INDEX.md) - Document navigation

---

**Document Version**: 1.0  
**Last Updated**: 2026-05-23  
**Total Stories**: ~95  
**Status**: ✅ Final - Ready for Sprint Planning

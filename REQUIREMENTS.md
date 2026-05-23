# Guardian Parent App - Requirements Document

> **Related Documents**: [INDEX.md](INDEX.md) | [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) | [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md) | [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md)  
> **Last Updated**: 2026-05-23  
> **Status**: Ready for Development (with full bidirectional cross-references)  

---

## 🔗 How to Use This Document

**All requirements include explicit cross-references** to user stories, API endpoints, and database tables:

- **FR-X.Y**: Each functional requirement shows which [US-XXX-MVP](#) user story implements it, which [EP-X](#) API endpoint satisfies it, and which [TB-X](#) database table stores the data
- **NFR-X.Y**: Each non-functional requirement references the APIs and tables it affects

**Quick Example**: Look at FR-1.1 (Guardian registration):
- ✅ **Implemented by**: [US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp-parent-registration)
- ✅ **API Contract**: [EP-1](TECH_ARCHITECTURE.md#ep-1-register-guardian)
- ✅ **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

**For a Complete Mapping**: See [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md) for all requirements, stories, APIs, and tables organized by feature.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Scope & Constraints](#scope--constraints)
3. [Feature Requirements by Phase](#feature-requirements-by-phase)
   - **Phase 1 (MVP): Core Features**
     - [1. Authentication & Guardian Management](#1-authentication--guardian-management)
     - [2. Sync & Offline Functionality](#2-sync--offline-functionality)
     - [3. Dashboard & Student Profile](#3-dashboard--student-profile)
     - [4. School Information & Multi-Tenancy](#4-school-information--multi-tenancy)
     - [5. Attendance Intelligence](#5-attendance-intelligence)
     - [6. Class Routine](#6-class-routine)
     - [7. Daily Class Activity](#7-daily-class-activity)
     - [8. Homework & Assignment Tracker](#8-homework--assignment-tracker)
     - [9. Examination & Result Management](#9-examination--result-management)
     - [10. Notifications](#10-notifications-core-implementation)
   - **Phase 2 (Enhanced): Engagement Features**
     - [11. Behavior & Discipline Analysis](#11-behavior--discipline-analysis)
     - [12. Unified Performance Scorecard](#12-unified-performance-scorecard)
     - [13. Smart Learning Progress Tracking](#13-smart-learning-progress-tracking)
     - [14. Fee Management](#14-fee-management)
     - [15. Notice Board](#15-notice-board)
     - [16. Event Management](#16-event-management)
     - [17. Holiday Calendar](#17-holiday-calendar)
     - [18. Issue Messaging System](#18-issue-messaging-system)
     - [19. Parent-Teacher Meeting (PTM)](#19-parent-teacher-meeting-ptm)
     - [20. Teacher Appointment](#20-teacher-appointment)
     - [21. Tomorrow's Due Dashboard](#21-tomorrows-due-dashboard)
     - [22. Phone Directory](#22-phone-directory)
   - **Phase 3 (Health & Growth): Wellness Features**
     - [23. Health & Growth Monitoring](#23-health--growth-monitoring)
     - [24. Vaccination Record](#24-vaccination-record)
     - [25. Daily Parent Tips](#25-daily-parent-tips)
     - [26. Classroom Snapshots](#26-classroom-snapshots)
     - [27. Advanced Calendar Integration](#27-advanced-calendar-integration)
4. [System-Wide Non-Functional Requirements](#system-wide-non-functional-requirements)
5. [Data Classification & Lifecycle](#data-classification--lifecycle)

---

## Executive Summary

**Project**: Guardian Parent App  
**Type**: Mobile-first SaaS platform  
**Target Market**: Bangladesh schools  
**Platform**: Flutter (Android 8+, iOS 13+)  
**Scope**: Parent mobile app only (not school backend)  

### **Core Problem Solved**
Parents currently lack visibility into their child's school performance, often missing important academic updates that get lost in WhatsApp group chats. Schools lack an effective channel for parent engagement.

### **Solution**
An offline-first mobile app that provides real-time updates on academic progress, attendance, homework, exam results, behavior, health, and school activities with intelligent notifications.

### **Success Metrics**
- ✅ Daily active users: 50%+ of registered parents
- ✅ Session length: 5+ minutes average
- ✅ Feature adoption: 80%+ of key features used monthly
- ✅ App store rating: 4.0+
- ✅ Uptime: 99%
- ✅ User retention: 70%+ after 30 days

### **Key Numbers**
| Metric | Target |
|--------|--------|
| **Initial Users** | 5,000 guardians |
| **Scaled Users** | 10,000 guardians |
| **API Response Time** | ≤ 120 seconds max |
| **Uptime** | 99% (52.6 min downtime/month) |
| **Features** | 28 total (9 MVP, 12 Phase 2, 7 Phase 3) |
| **User Stories** | ~95 across all phases |
| **API Endpoints** | ~45-50 endpoints |
| **SQLite Tables** | ~25-30 local storage tables |

---

## Scope & Constraints

### **In Scope**
- ✅ Parent mobile app (Flutter)
- ✅ Offline-first functionality with SQLite
- ✅ Sync with backend API (.NET Core)
- ✅ Multi-school support
- ✅ Multi-child profile management
- ✅ Push notifications (FCM)
- ✅ All 28 features (phased)

### **Out of Scope**
- ❌ School operations backend (.NET Core backend is out of scope for this documentation)
- ❌ School administrator portal
- ❌ Teacher functionality
- ❌ Backend database design (PostgreSQL, MongoDB, etc.)
- ❌ Web-based parent portal
- ❌ Video streaming / classroom capture

### **Constraints**
- **Technology**: Flutter only (not React Native, not native)
- **Local Database**: SQLite (not Hive, not Firebase offline)
- **Backend Communication**: REST JSON APIs only
- **Language**: Bangla (primary), English (secondary)
- **Device**: Android 8.0+ and iOS 13.0+
- **Bandwidth**: Optimized for 3G networks
- **Data**: No GDPR compliance required; no educational data standards required
- **Payment**: PCI-DSS 3.2.1 required for bKash integration

---

## Feature Requirements by Phase

### Phase 1 (MVP): Core Features

**Duration**: Weeks 1-12  
**Stories**: 40  
**Users**: 5,000  

#### **1. Authentication & Guardian Management**

**Overview**: Guardian authentication with two types (Key Guardian designated by school, Additional Guardian self-registered). OTP verification only during child-linking, no OTP on login.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-1-authentication--account-management)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#authentication-endpoints)**

**Functional Requirements**:
- **FR-1.1: Guardian registration (email + password)**
  - Email format validation, uniqueness check
  - Password: min 8 chars, 1 uppercase, 1 lowercase, 1 digit
  - Bcrypt hash storage (≥10 salt rounds)
  - **Implemented by**: [US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp-parent-registration) | **API**: [EP-1](TECH_ARCHITECTURE.md#ep-1-register-guardian) | **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

- **FR-1.2: Guardian login (email + password, no OTP)**
  - JWT token issued (30-day expiry, default 24h)
  - 3 failed attempts → 15-minute lockout
  - Revocation status check
  - **Implemented by**: [US-002-MVP](USER_STORIES_PERSONAS.md#us-002-mvp-guardian-login) | **API**: [EP-2](TECH_ARCHITECTURE.md#ep-2-guardian-login) | **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

- **FR-1.3: Key Guardian detection**
  - Email matches student_master.key_guardian_email → Key Guardian flow
  - Email doesn't match → Additional Guardian flow (in-app approval)
  - **Implemented by**: [US-003-MVP](USER_STORIES_PERSONAS.md#us-003-mvp-key-guardian-add-first-child-with-otp), [US-004-MVP](USER_STORIES_PERSONAS.md#us-004-mvp-additional-guardian-request-access) | **API**: [EP-3](TECH_ARCHITECTURE.md#ep-3-send-otp-child-linking), [EP-4](TECH_ARCHITECTURE.md#ep-4-verify-otp--link-child)

- **FR-1.4: OTP verification (Key Guardian only)**
  - 6-digit OTP sent to registered phone
  - 10-minute expiry, max 3 wrong attempts
  - 30-second resend cooldown, max 3 resends
  - **Implemented by**: [US-003-MVP](USER_STORIES_PERSONAS.md#us-003-mvp-key-guardian-add-first-child-with-otp) | **API**: [EP-3](TECH_ARCHITECTURE.md#ep-3-send-otp-child-linking), [EP-4](TECH_ARCHITECTURE.md#ep-4-verify-otp--link-child)

- **FR-1.5: Child linking (Key Guardian)**
  - Student ID + DOB validation
  - OTP verification
  - Device registration
  - Initial sync triggered (blocking)
  - **Implemented by**: [US-003-MVP](USER_STORIES_PERSONAS.md#us-003-mvp-key-guardian-add-first-child-with-otp) | **API**: [EP-4](TECH_ARCHITECTURE.md#ep-4-verify-otp--link-child), [EP-5](TECH_ARCHITECTURE.md#ep-5-register-device), [EP-6](TECH_ARCHITECTURE.md#ep-6-initial-sync) | **Database**: [TB-3](TECH_ARCHITECTURE.md#tb-3-guardian-children-link-table), [TB-4](TECH_ARCHITECTURE.md#tb-4-student-profile-table)

- **FR-1.6: Child linking (Additional Guardian)**
  - Student ID + DOB validation
  - In-app approval request to Key Guardian
  - 48-hour expiry, 24-hour re-submit wait on deny
  - Sync triggered on approval
  - **Implemented by**: [US-004-MVP](USER_STORIES_PERSONAS.md#us-004-mvp-additional-guardian-request-access), [US-005-MVP](USER_STORIES_PERSONAS.md#us-005-mvp-key-guardian-approvdeny-access-request) | **Database**: [TB-3](TECH_ARCHITECTURE.md#tb-3-guardian-children-link-table)

- **FR-1.7: Device binding**
  - One account = one active device
  - New device login locks old device
  - Old device logs out on next sync
  - **Implemented by**: [US-006-MVP](USER_STORIES_PERSONAS.md#us-006-mvp-new-device-login-warning) | **API**: [EP-2](TECH_ARCHITECTURE.md#ep-2-guardian-login), [EP-5](TECH_ARCHITECTURE.md#ep-5-register-device), [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync) | **Database**: [TB-2](TECH_ARCHITECTURE.md#tb-2-devices-table)

- **FR-1.8: Additional Guardian revocation (Key Guardian)**
  - Mark as revoked
  - Immediate logout detected on next delta sync (within 10 minutes)
  - Additional Guardian shown in-app message on next sync: "Your access has been revoked. Contact the key guardian." (no push notification)
  - **Implemented by**: [US-007-MVP](USER_STORIES_PERSONAS.md#us-007-mvp-revoke-additional-guardian-access) | **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table), [TB-3](TECH_ARCHITECTURE.md#tb-3-guardian-children-link-table)

- **FR-1.9: Forgot password**
  - Email reset link (30-min expiry, single-use)
  - All sessions invalidated on reset
  - Confirmation message shown
  - **Implemented by**: [US-008-MVP](USER_STORIES_PERSONAS.md#us-008-mvp-forgot-password) | **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

**Non-Functional Requirements**:
- **NFR-1.1: Performance**
  - Registration: ≤ 2 seconds | **API**: [EP-1](TECH_ARCHITECTURE.md#ep-1-register-guardian)
  - Login: ≤ 2 seconds | **API**: [EP-2](TECH_ARCHITECTURE.md#ep-2-guardian-login)
  - OTP send: ≤ 3 seconds | **API**: [EP-3](TECH_ARCHITECTURE.md#ep-3-send-otp-child-linking)
  - OTP verify: ≤ 2 seconds | **API**: [EP-4](TECH_ARCHITECTURE.md#ep-4-verify-otp--link-child)
  - Device register: ≤ 1 second | **API**: [EP-5](TECH_ARCHITECTURE.md#ep-5-register-device)

- **NFR-1.2: Security**
  - HTTPS only (TLS 1.2+)
  - Passwords: Bcrypt with ≥10 salt rounds | **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)
  - JWT: HS256 or RS256
  - Rate limiting: 5 attempts / 15 min per email
  - OTP: 6-digit, 10-min expiry, 3-attempt limit, SMS delivery 98% | **API**: [EP-3](TECH_ARCHITECTURE.md#ep-3-send-otp-child-linking), [EP-4](TECH_ARCHITECTURE.md#ep-4-verify-otp--link-child)

- **NFR-1.3: Reliability**
  - SMS delivery: 98% success within 30 seconds
  - Email delivery: 99% success within 5 minutes
  - No session loss during network interruption | **User Stories**: [US-011-MVP](USER_STORIES_PERSONAS.md#us-011-mvp-view-data-offline-after-sync), [US-012-MVP](USER_STORIES_PERSONAS.md#us-012-mvp-handle-sync-failures-gracefully)

**Acceptance Criteria**:
- ✅ User can register with valid credentials
- ✅ Duplicate email prevented
- ✅ Weak password rejected
- ✅ Key guardian receives OTP on phone
- ✅ Correct OTP links child; wrong OTP rejected (max 3 attempts)
- ✅ Additional guardian request approved/denied properly
- ✅ Revoked guardian immediately logged out
- ✅ New device login locks old device
- ✅ Password reset invalidates all sessions

---

#### **2. Sync & Offline Functionality**

**Overview**: Delta sync architecture with local SQLite. 10-minute background sync when online. Full initial sync on first login. All read features work offline.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-2-sync--offline-functionality)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#sync-endpoints)**  
**[See Database Schema →](TECH_ARCHITECTURE.md#sqlite-database-schema)**

**Functional Requirements**:
- **FR-2.1: Data classification**
  - Academic-year data: Exams, homework, class activity, class routine (current year only)
  - Continuous data: Vaccination, health, behavior, student profile, attendance (full history)
  - Automatic purge of academic-year data > 1 year old
  - Current + previous year academic data retained
  - **Implemented by**: [US-013-MVP](USER_STORIES_PERSONAS.md#us-013-mvp-auto-purge-old-academic-year-data) | **Database**: [TB-6](TECH_ARCHITECTURE.md#tb-6-attendance-table), [TB-7](TECH_ARCHITECTURE.md#tb-7-homework-table), [TB-8](TECH_ARCHITECTURE.md#tb-8-exam-table), [TB-10](TECH_ARCHITECTURE.md#tb-10-class-activity-table), [TB-11](TECH_ARCHITECTURE.md#tb-11-class-routine-table)

- **FR-2.2: Initial full sync**
  - Trigger: After child linking
  - Blocking sync with progress bar
  - All school_info, student_profile, academic_year_data, continuous_data downloaded
  - SQLite database created locally
  - School branding applied
  - FCM topics subscribed
  - Timeout: 3 failures → offline mode allowed, retry on connection
  - **Implemented by**: [US-009-MVP](USER_STORIES_PERSONAS.md#us-009-mvp-initial-full-sync-on-first-child-link) | **API**: [EP-6](TECH_ARCHITECTURE.md#ep-6-initial-sync) | **Database**: [TB-4](TECH_ARCHITECTURE.md#tb-4-student-profile-table), [TB-5](TECH_ARCHITECTURE.md#tb-5-sync-metadata-table), [TB-12](TECH_ARCHITECTURE.md#tb-12-school-info-table)

- **FR-2.3: Background 10-minute delta sync**
  - Runs every 10 minutes when online
  - Per-school sync requests (one request per school)
  - Only changed data fetched (based on data_change_log)
  - Response merged into local SQLite
  - sync_metadata updated with last_sync_datetime
  - "Last synced: just now" shown in UI
  - Retry on failure at next cycle
  - **Implemented by**: [US-010-MVP](USER_STORIES_PERSONAS.md#us-010-mvp-background-delta-sync-10-minute-cycle) | **API**: [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync) | **Database**: [TB-5](TECH_ARCHITECTURE.md#tb-5-sync-metadata-table)

- **FR-2.4: Offline experience**
  - All read features work offline (no edits)
  - "Offline" indicator shown
  - Data staleness clearly displayed (< 24h, 24-48h warning, > 7 days alert)
  - Automatic sync on connection return
  - **Implemented by**: [US-011-MVP](USER_STORIES_PERSONAS.md#us-011-mvp-view-data-offline-after-sync) | **Database**: [TB-5](TECH_ARCHITECTURE.md#tb-5-sync-metadata-table)

- **FR-2.5: Revocation detection**
  - Sync response includes revocation_status flag
  - Revoked guardian: Data cleared, logout forced
  - Message: "Your access has been revoked. Contact the key guardian."
  - Detection within 10 minutes via delta sync (no FCM push to Additional Guardian)
  - **Implemented by**: [US-007-MVP](USER_STORIES_PERSONAS.md#us-007-mvp-revoke-additional-guardian-access) | **API**: [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync)

- **FR-2.6: Device locking**
  - New device login marks old device_id as locked
  - Old device next sync: 403 error, forced logout
  - Message: "You logged in from another device. Session ended."
  - **Implemented by**: [US-006-MVP](USER_STORIES_PERSONAS.md#us-006-mvp-new-device-login-warning) | **API**: [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync) | **Database**: [TB-2](TECH_ARCHITECTURE.md#tb-2-devices-table)

- **FR-2.7: Multi-school sync**
  - Per-school sync cycles (independent)
  - UI shows: "Last synced at School A: 2 min ago, School B: 5 min ago"
  - No global bottleneck
  - **Implemented by**: [US-021-MVP](USER_STORIES_PERSONAS.md#us-021-mvp-support-multiple-schools--children) | **API**: [EP-6](TECH_ARCHITECTURE.md#ep-6-initial-sync), [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync) | **Database**: [TB-5](TECH_ARCHITECTURE.md#tb-5-sync-metadata-table)

- **FR-2.8: Academic year transition**
  - Automatic purge at year boundary
  - Continuous data retained
  - UI updated to show current academic year
  - **Implemented by**: [US-013-MVP](USER_STORIES_PERSONAS.md#us-013-mvp-auto-purge-old-academic-year-data)

**Non-Functional Requirements**:
- **NFR-2.1: Performance**
  - Initial sync response: ≤ 120 seconds | **API**: [EP-6](TECH_ARCHITECTURE.md#ep-6-initial-sync)
  - Delta sync response: ≤ 5 seconds | **API**: [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync)
  - Local data query: ≤ 200 ms
  - Sync merge: ≤ 500 ms

- **NFR-2.2: Bandwidth**
  - Initial sync: 500 KB - 2 MB (gzip: 250 KB - 1 MB) | **API**: [EP-6](TECH_ARCHITECTURE.md#ep-6-initial-sync)
  - Delta sync: 10-50 KB typical (gzip: 5-25 KB) | **API**: [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync)
  - Monthly usage: ~43 MB compressed (8 hrs/day, 4,320 cycles)

- **NFR-2.3: Storage**
  - Local SQLite DB: < 500 MB per device | **Database**: All [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table) through [TB-13](TECH_ARCHITECTURE.md#tb-13-school-contacts-table)
  - Works on 2GB+ RAM devices

- **NFR-2.4: Reliability**
  - Sync success: > 99% on stable connection | **API**: [EP-6](TECH_ARCHITECTURE.md#ep-6-initial-sync), [EP-7](TECH_ARCHITECTURE.md#ep-7-delta-sync)
  - Data consistency: Eventual < 10 minutes
  - No data loss on failure
  - Retry: Exponential backoff up to 10 minutes | **User Stories**: [US-012-MVP](USER_STORIES_PERSONAS.md#us-012-mvp-handle-sync-failures-gracefully)

- **NFR-2.5: Scalability**
  - Per-school sync cycles (no global bottleneck) | **Database**: [TB-5](TECH_ARCHITECTURE.md#tb-5-sync-metadata-table)
  - Support 1,000+ schools at scale
  - Support 10,000+ concurrent active syncs

**Acceptance Criteria**:
- ✅ First login triggers initial sync with progress bar
- ✅ Full data downloaded and stored in SQLite
- ✅ 10-minute sync runs automatically every 10 minutes
- ✅ Only changed data fetched (delta)
- ✅ Offline mode shows cached data with staleness indicator
- ✅ Connection return triggers auto-sync
- ✅ Revoked guardian force-logged out on next sync
- ✅ Device locked on new login; old device logs out
- ✅ Multi-school: Separate sync cycles, independent data
- ✅ Academic year boundary: Old data purged, continuous data retained

---

#### **3. Dashboard & Student Profile**

**Overview**: Quick overview of child's performance with school branding. Multi-child switching. Profile information display.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-3-dashboard--profile)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#dashboard-endpoint)**

**Functional Requirements**:
- FR-3.1: Dashboard display
  - Student name, photo, class, section
  - School branding: Logo, colors, name
  - Quick stats:
    - Attendance % (last 30 days)
    - Next homework due (with subject)
    - Upcoming exam (date, subject)
    - Alert badge if low attendance (< 75%)
  - Last synced timestamp
  - Loads from local cache (instant)
- FR-3.2: Multi-child profile switching
  - Dropdown/carousel showing all linked children
  - Tap child loads dashboard instantly (< 500ms)
  - Last selected child remembered
  - All data reloads for selected child
- FR-3.3: Detailed student profile
  - Name, photo, DOB, class, section, curriculum, session
  - Guardian info: Name, relationship
  - Emergency contacts: Name, phone
  - Address from student master
  - Read-only (edits managed by school)
- FR-3.4: Additional guardian management (Key Guardian)
  - List of linked additional guardians
  - Per guardian: Name, email, relationship, linked date
  - Revoke button with confirmation
  - Call FR-1.8 (revoke endpoint)

**Non-Functional Requirements**:
- NFR-3.1: Performance
  - Dashboard load: ≤ 1 second (local cache)
  - Child switch: ≤ 500 ms
  - Stats calculation: ≤ 200 ms
- NFR-3.2: UI/UX
  - Mobile-responsive (4.5" - 6.5" phones)
  - Fast child switching
  - Visual hierarchy clear
  - Touch-friendly buttons (48x48 dp minimum)

**Acceptance Criteria**:
- ✅ Dashboard loads with student name, photo, school logo
- ✅ Quick stats displayed accurately
- ✅ Child switching instant and correct
- ✅ Profile shows all details correctly
- ✅ Key guardian can see and revoke additional guardians
- ✅ Design responsive on all screen sizes
- ✅ Offline: Cached dashboard displayed

---

#### **4. School Information & Multi-Tenancy**

**Overview**: School discovery, multi-tenancy isolation, school branding, contact directory.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-4-school-information--multi-tenancy)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#school-endpoints)**

**Functional Requirements**:
- FR-4.1: School discovery
  - Search school by name / location / code
  - List matching schools with logo, address, contact
  - Select school to proceed
  - School details displayed
- FR-4.2: School branding
  - Logo, primary color, secondary color, accent color
  - Applied to entire UI
  - Downloaded in initial sync, cached locally
  - Updated in each sync
- FR-4.3: Contact directory
  - Principal: Name, phone, email
  - Administration: Staff names, phones
  - Class teachers: All teachers with contact info
  - Support staff: Counselor, support team
  - Tap phone → dial app opens
  - Tap email → email app opens
- FR-4.4: Multi-tenancy isolation
  - RLS enforced at database level
  - Guardian only sees children from linked schools
  - Cross-school contamination prevented
  - Backend validates school_id on every request

**Non-Functional Requirements**:
- NFR-4.1: Data isolation
  - RLS enforced: No cross-school leakage
  - Tenant isolation: All layers
  - Backend validation: Every request
  - Audit: Cross-school attempts logged
- NFR-4.2: Scalability
  - Support 1,000+ schools
  - No performance degradation
  - Per-school data independence
- NFR-4.3: Performance
  - Directory load: ≤ 500 ms
  - School search: ≤ 1 second
  - School switch: ≤ 300 ms

**Acceptance Criteria**:
- ✅ School searched and found correctly
- ✅ School branding applied throughout app
- ✅ Contact directory complete and callable
- ✅ Guardian can't see other schools' data
- ✅ Multi-school: Data isolated per school
- ✅ Offline: Directory cached and accessible

---

#### **5. Attendance Intelligence**

**Overview**: Daily attendance tracking, monthly summaries, alerts for low/irregular attendance.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-5-attendance-intelligence)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#attendance-endpoints)**

**Functional Requirements**:
- FR-5.1: Daily attendance calendar
  - 30-day calendar with color coding:
    - Green = Present
    - Red = Absent
    - Yellow = Leave
    - Orange = Half-day
  - Tap date shows remarks
  - Monthly attendance % calculated
- FR-5.2: Monthly attendance summary
  - Calculate: (Present + Half-days*0.5) / Total days * 100
  - Show: Percentage, day counts
  - Trend: vs. previous month, vs. last 3 months average
- FR-5.3: Attendance analytics
  - Daily average: Count per weekday
  - Weekly trend: Last 4 weeks (bar chart)
  - Monthly trend: Last 6 months (line chart)
  - Class average: Anonymized comparison
- FR-5.4: Low attendance alert
  - Threshold: < 75% (configurable by school)
  - Push notification + in-app notification
  - Red badge on dashboard
  - Frequency: Once per day (if condition persists)
  - Message: "[Child Name]'s attendance dropped to X%"
- FR-5.5: Unauthorized absence alert
  - Pattern: 2+ consecutive absences without 'leave' mark
  - Alert: "Unauthorized absence noticed. Please contact school."
  - Red indicator on calendar
- FR-5.6: Attendance details per date
  - Status, remarks, duration (if half-day)
  - Check in/out times (if available)

**Non-Functional Requirements**:
- NFR-5.1: Performance
  - Load calendar: ≤ 500 ms
  - Calculate percentage: ≤ 100 ms
  - Render charts: ≤ 1 second
  - Alert detection: ≤ 200 ms on sync
- NFR-5.2: Accuracy
  - Percentage calculation: ±0 error margin
  - No rounding unless specified
- NFR-5.3: Reliability
  - Alert delivery: 99% success
  - Consistent calculations

**Acceptance Criteria**:
- ✅ Attendance calendar shows 30 days with colors
- ✅ Monthly summary displays percentage correctly
- ✅ Low attendance < 75% shows alert
- ✅ Unauthorized absences flagged
- ✅ Trend analytics displayed
- ✅ Offline: Last synced attendance shown
- ✅ Calculations 100% accurate

---

#### **6. Class Routine**

**Overview**: Weekly class schedule with subjects, timings, and assigned teachers.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-9-class-routine)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#routine-endpoints)**

**Functional Requirements**:
- FR-6.1: Weekly timetable display
  - Monday-Friday (or as per school)
  - Per time slot: Subject name, teacher name, room/location
  - Today's schedule highlighted
  - 12-hour or 24-hour format (per school config)
- FR-6.2: Teacher assignment
  - Teacher name for each subject
  - Tap teacher: Show contact details from directory
  - Different routines for different sections
- FR-6.3: Routine updates
  - Synced via background sync
  - Changes reflected immediately
  - Update notification (if significant changes)
- FR-6.4: Offline access
  - Cached routine displayed
  - "Last updated: X minutes ago" shown

**Acceptance Criteria**:
- ✅ Timetable displayed correctly with all slots filled
- ✅ Tap teacher shows contact information
- ✅ Today's schedule visually distinguished
- ✅ Routine updates on sync
- ✅ Works offline with cached data

---

#### **7. Daily Class Activity**

**Overview**: Daily updates about classroom topics, activities, and homework.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-8-daily-class-activity)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#activity-endpoints)**

**Functional Requirements**:
- FR-7.1: Daily activity entry
  - Date, subject, topics taught, chapters covered, activities, remarks
  - Card view (one per day, recent first)
  - 7-day view by default
  - Load older functionality
- FR-7.2: Activity details
  - Topics: List of topics covered
  - Chapters: Chapter numbers/names
  - Activities: Classroom activities, projects, experiments
  - Remarks: Teacher remarks
  - Related homework: Link to homework assigned same day
- FR-7.3: Activity filtering
  - By subject
  - By week (jump to specific week)
  - By date range (search)
- FR-7.4: Offline access
  - Last 7 days cached
  - Offline indicator shown

**Acceptance Criteria**:
- ✅ Activity feed shows last 7 days
- ✅ Tap activity shows all details
- ✅ Filter by subject works
- ✅ Load older loads previous weeks
- ✅ Works offline with cached data

---

#### **8. Homework & Assignment Tracker**

**Overview**: Complete homework management including assignment, submission, evaluation, deadline tracking with teacher feedback.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-6-homework--assignments)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#homework-endpoints)**

**Functional Requirements**:
- FR-8.1: Homework list display
  - Subject, title, description, assigned date, due date
  - Status: Assigned, Submitted, Evaluated, Pending, Late, Overdue
  - Teacher marks, grade, feedback (if evaluated)
  - Sort: By due date (default), by subject, by status
  - Filter: By subject, by status, by date range
- FR-8.2: Homework status tracking
  - Assigned: Show on assignment date + due date
  - Submitted: Mark when student submits
  - Pending: Show if evaluation pending
  - Evaluated: Show marks, grade, feedback
  - Late: If submitted after due date
  - Overdue: If due date passed without submission
- FR-8.3: Teacher feedback & scoring
  - Marks/score (out of total)
  - Grade (A/B/C/D/F)
  - Remarks (teacher comments)
  - Highlight: Excellent (A/90+) green, Needs improvement (D/F/<60) red
- FR-8.4: Deadline reminders
  - 1 day before due date
  - Push notification: "[Subject] homework due tomorrow"
  - In-app: "Due tomorrow" label
  - Delivery success: > 99%
- FR-8.5: Subject-wise summary
  - Total homework, completed, pending, late
  - Trend: Late submissions per subject
- FR-8.6: Mark as viewed
  - Track read status
  - Unread indicator for new homework
- FR-8.7: Monthly summary (Phase 2)
  - Total assigned/submitted/pending
  - Completion rate per subject
  - Late submission %

**Non-Functional Requirements**:
- NFR-8.1: Performance
  - List load: ≤ 500 ms
  - Filter/sort: ≤ 300 ms
  - Feedback sync: ≤ 1 second
- NFR-8.2: Notifications
  - Reminder: Exactly 24h before due date
  - Delivery: 99% success
  - Retry: Max 3 attempts

**Acceptance Criteria**:
- ✅ All homework listed with subject, due date, status
- ✅ Homework due tomorrow gets reminder
- ✅ Teacher evaluations display marks, grade, feedback
- ✅ Filter by subject/status works
- ✅ Late submissions marked clearly
- ✅ Works offline with cached data

---

#### **9. Examination & Result Management**

**Overview**: Exam scheduling, result publication, performance analytics, and result history.

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-7-exams--results)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#exam-endpoints)**

**Functional Requirements**:
- FR-9.1: Exam schedule
  - Exam type, date, time, subject, duration, instructions
  - Upcoming exams shown chronologically
  - Calendar: Exam dates highlighted
  - Reminder: 1 day before exam
- FR-9.2: Exam results
  - Subject, marks obtained/total, grade, percentile
  - Show on exam date + 1-2 days
  - Subject-wise breakdown
- FR-9.3: Comparative analytics
  - Class average (anonymized)
  - Student percentile ("Top 25% for Math")
  - Performance band (Advanced/Proficient/Developing)
  - Subject-wise comparison
- FR-9.4: Performance trend analysis
  - Subject trend: Line graph of marks across exams
  - Exam trend: Radar chart across subjects
  - Progress: Improvement/decline indicators
  - Time range: Last 3 terms or 1 year
- FR-9.5: Exam result history
  - All results available
  - Filter: By exam type, subject, date range
  - Download: Export result as PDF (Phase 2)
- FR-9.6: Exam prep resources (Phase 2)
  - Syllabus: Topic-wise breakdown
  - Previous papers (if provided)
  - Study materials: Links to resources

**Acceptance Criteria**:
- ✅ Exam schedule shows all upcoming exams
- ✅ Results published on exam date
- ✅ Marks, grades, percentile shown
- ✅ Trend analytics displayed
- ✅ Class average and ranking shown (if enabled)
- ✅ Works offline with cached data

---

#### **10. Notifications (Core Implementation)**

**Overview**: Push notifications for key events (homework due, exams, attendance alerts) with user preferences. **All push notifications are delivered to Key Guardian only. Additional Guardian receives no push notifications and accesses data via app sync only.**

**[See User Stories →](USER_STORIES_PERSONAS.md#epic-10-notifications)**  
**[See API Contracts →](TECH_ARCHITECTURE.md#notification-endpoints)**

**Functional Requirements**:
- FR-10.1: Push notifications for events (Key Guardian only)
  - **Recipient**: Key Guardian only — Additional Guardian receives no push notifications
  - Homework due tomorrow
  - Exam announced / scheduled
  - Low attendance alert (< 75%)
  - Behavioral concerns
  - Assignment evaluated
  - School announcements (important)
  - Fee due (Phase 2)
  - Event announcements
- FR-10.2: Notification delivery (Key Guardian only)
  - Firebase Cloud Messaging (FCM) for push to Key Guardian devices only
  - Additional Guardian: No FCM push; reads updated data via app sync only
  - Tap notification opens relevant section
  - Notification context (which child if multiple)
  - Delivery tracking
- FR-10.3: Notification preferences (Key Guardian only)
  - Enable/disable per category
  - Set quiet hours (e.g., 9 PM - 8 AM)
  - Frequency control (e.g., once per day for repeat alerts)
  - Settings synced across Key Guardian devices
  - Defaults: All enabled, 9 PM - 8 AM quiet
- FR-10.4: Notification history (Key Guardian only)
  - Notification center view (reverse chronological)
  - Mark as read/unread
  - Search notifications
  - Clear all option (optional)
- FR-10.5: Smart notification rules (Key Guardian only)
  - Deduplication: No duplicates for same event
  - Batching: Multiple items into one notification
  - Aggregation: "2 homework + 1 exam result available"
  - Cooldown: Max 1 low-attendance alert per day

**Non-Functional Requirements**:
- NFR-10.1: Reliability
  - Push delivery: 99% success (FCM-level)
  - Latency: < 5 seconds from event to push
  - Retries: Max 3 on delivery failure
- NFR-10.2: Scalability
  - Support millions of concurrent notifications to Key Guardians
  - Rate limiting: Max 10 notifications per hour per Key Guardian
- NFR-10.3: Privacy
  - Content not logged on server
  - User preferences always respected
  - No profiling based on notification behavior

**Acceptance Criteria**:
- ✅ Push notifications delivered to Key Guardian only
- ✅ Additional Guardian receives no push notifications
- ✅ Homework due tomorrow gets push notification (Key Guardian)
- ✅ Low attendance alert sent (Key Guardian)
- ✅ Exam announced notification received (Key Guardian)
- ✅ Tap notification opens correct section
- ✅ Quiet hours respected (no notifications)
- ✅ Duplicate notifications prevented
- ✅ Key Guardian can disable/enable notification categories

---

### Phase 2 (Enhanced): Engagement Features

**Duration**: Weeks 13-20  
**Stories**: 30-35  
**Users**: 5,000+  

**Note**: Each Phase 2 feature follows same structure as Phase 1 — Overview, Functional Requirements, Non-Functional Requirements, Acceptance Criteria.  
**[See User Stories →](USER_STORIES_PERSONAS.md#phase-2-enhanced-backlog)** | **[See API Contracts & DB Tables →](TECH_ARCHITECTURE.md#sqlite-database-schema)**

---

#### **11. Behavior & Discipline Analysis**

> Full specifications to be completed in next iteration.

---

#### **12. Unified Performance Scorecard**

> Full specifications to be completed in next iteration.

---

#### **13. Smart Learning Progress Tracking**

> Full specifications to be completed in next iteration.

---

#### **14. Fee Management**

> Full specifications to be completed in next iteration.

---

#### **15. Notice Board**

> Full specifications to be completed in next iteration.

---

#### **16. Event Management**

> Full specifications to be completed in next iteration.

---

#### **17. Holiday Calendar**

> Full specifications to be completed in next iteration.

---

#### **18. Issue Messaging System**

> Full specifications to be completed in next iteration.

---

#### **19. Parent-Teacher Meeting (PTM)**

> Full specifications to be completed in next iteration.

---

#### **20. Teacher Appointment**

> Full specifications to be completed in next iteration.

---

#### **21. Tomorrow's Due Dashboard**

> Full specifications to be completed in next iteration.

---

#### **22. Phone Directory**

> Full specifications to be completed in next iteration.

---

### Phase 3 (Health & Growth): Wellness Features

**Duration**: Weeks 21-28  
**Stories**: 15-20  
**Users**: 5,000+  

**Note**: Each Phase 3 feature follows same structure as Phase 1 — Overview, Functional Requirements, Non-Functional Requirements, Acceptance Criteria.  
**[See User Stories →](USER_STORIES_PERSONAS.md#phase-3-health--growth-backlog)** | **[See API Contracts & DB Tables →](TECH_ARCHITECTURE.md#sqlite-database-schema)**

---

#### **23. Health & Growth Monitoring**

> Full specifications to be completed in next iteration.

---

#### **24. Vaccination Record**

> Full specifications to be completed in next iteration.

---

#### **25. Daily Parent Tips**

> Full specifications to be completed in next iteration.

---

#### **26. Classroom Snapshots**

> Full specifications to be completed in next iteration.

---

#### **27. Advanced Calendar Integration**

> Full specifications to be completed in next iteration.

---

## System-Wide Non-Functional Requirements

### **Performance Requirements**

| Metric | Target | Justification |
|--------|--------|---------------|
| App startup time | ≤ 3 seconds | User tolerance for first load |
| Dashboard load | ≤ 1 second | Most common action |
| Page/module load | ≤ 500 ms | User experience on navigation |
| **API response time** | **≤ 120 seconds max** | **User requirement** |
| Sync response time | ≤ 10s (4G), ≤ 30s (3G) | Network dependent |
| Search results | ≤ 500 ms | Quick filtering |
| Chart rendering | ≤ 1 second | Analytics load |
| Local data query | ≤ 200 ms | SQLite performance |
| Image load | ≤ 1 second | UI responsiveness |

### **Reliability & Availability**

| Metric | Target | Notes |
|--------|--------|-------|
| **Platform uptime** | **99%** | 52.6 minutes downtime/month allowed |
| Sync success rate | > 99% | On stable connection |
| Push notification delivery | > 99% | FCM-level guarantee |
| Data consistency | Eventual < 10 min | Acceptable for education context |
| Zero data loss | 100% | Database replication enforced |
| Backup strategy | Daily, 30-day retention | Disaster recovery |

### **Scaling & Capacity**

| Metric | Initial | Scale | Notes |
|--------|---------|-------|-------|
| **Active Users** | **5,000** | **10,000** | Concurrent usage |
| Concurrent API requests | 50/sec | 100+/sec | Peak hour |
| Database connections | 200 pool | 500 pool | Connection pooling |
| Schools supported | 50+ | 1,000+ | Multi-tenancy |
| Users per school | 500-5,000 | 5,000-50,000 | Varies |
| Data size per user | ~50 MB | ~100 MB | 5-10 years history |

### **Security Requirements**

| Requirement | Standard | Details |
|-------------|----------|---------|
| **Encryption in Transit** | TLS 1.2+ | HTTPS for all endpoints |
| **Authentication** | JWT | 24-hour default, 30-day remember-me |
| **Authorization** | RLS + Backend validation | Database + API level |
| **Password Storage** | Bcrypt | ≥10 salt rounds |
| **OTP** | 6-digit SMS | 10-minute expiry, 3-attempt limit |
| **Device Binding** | One active per account | New device locks previous |
| **Rate Limiting** | Account lockout | 3 failed attempts → 15-minute lockout |
| **Payment Security** | **PCI-DSS 3.2.1** | **Required for bKash integration** |
| **Data Encryption** | At-rest (Phase 2+) | SQLite can be encrypted locally |
| **Session Timeout** | 24 hours | Default; 30 days with remember-me |
| **Audit Logging** | All sensitive operations | Access, modifications, deletions |

### **Privacy & Data Protection**

| Requirement | Standard | Details |
|-------------|----------|---------|
| User Consent | Explicit | Before data collection |
| Data Minimization | Only necessary | Collect what's needed |
| Data Retention | As per requirements | Purge old data per classification |
| Right to Deletion | On request | Account deletion removes all data |
| Breach Notification | 72 hours | Within 72h of discovery (if required) |
| Child Safety | Privacy-first | No public sharing, parental controls |
| GDPR Compliance | **NOT REQUIRED** | Bangladesh market focus |
| Educational Standards | **NOT REQUIRED** | Focus on utility, not compliance |

### **Device Support**

| Platform | Version | Notes |
|----------|---------|-------|
| **Android** | **8.0+ (API 26+)** | Tested on 2GB RAM minimum |
| **iOS** | **13.0+** | iPhone 6S and newer |
| Screen Sizes | 4.5" - 6.5" phones | Plus tablets (optional) |
| Network | 2G/3G/4G/5G | Optimized for 3G (Bangladesh) |
| Device Memory | 2GB+ RAM | Most common in target market |

### **Internationalization & Localization**

| Aspect | Standard | Details |
|--------|----------|---------|
| **Languages** | Bangla (primary), English | Both fully translated |
| **Date Format** | DD/MM/YYYY | Bangladesh standard |
| **Currency** | ৳ (BDT) | Bangladeshi Taka |
| **Time Format** | 12-hour (AM/PM) | User preference configurable |
| **Timezone** | UTC+6 | Bangladesh Standard Time |
| **Phone Format** | +880 with validation | Country-specific |
| **Regional Holidays** | Bangladesh public holidays | Integrated into calendar |

### **Offline Functionality**

| Aspect | Requirement | Details |
|--------|-------------|---------|
| Read Features | All work offline | No internet needed |
| Write Features | Blocked offline | Prevent conflicts |
| Data Caching | SQLite < 500 MB | Full storage |
| Offline Duration | 7+ days | Supported |
| Staleness Indicator | Always shown | User aware of data age |
| Auto-Sync Return | Immediate | Triggers on connection |
| Conflict Resolution | None needed | Read-only offline |

### **Accessibility**

| Standard | Level | Details |
|----------|-------|---------|
| **WCAG** | **2.1 AA minimum** | Best effort |
| Color Contrast | 4.5:1 for text | 3:1 for large text |
| Font Size | 16px minimum (body) | Scalable |
| Navigation | Keyboard support | Mobile keyboard |
| Screen Reader | Compatible | TalkBack, VoiceOver |
| Captions | Optional (Phase 3+) | Video content |

### **Monitoring & Operations**

| Component | Requirement | Details |
|-----------|-------------|---------|
| Error Tracking | Sentry/similar | Stack traces captured |
| Performance Monitoring | RUM data | User experience metrics |
| Analytics | Engagement tracking | Feature usage, funnels |
| Logging | Structured logs | 90-day retention |
| Alerting | Critical errors | Ops team notification |
| SLA Target | 99% uptime | < 5 min incident response |
| Backup | Daily, 30-day retention | Disaster recovery |

---

## Data Classification & Lifecycle

### **Data Types by Classification**

| Data Type | Classification | Lifecycle | Retention |
|-----------|----------------|-----------|-----------|
| Exam | Academic-year | Current + previous year | 2 years |
| Homework | Academic-year | Current + previous year | 2 years |
| Class activity | Academic-year | Current + previous year | 2 years |
| Class routine | Academic-year | Current year only | 1 year |
| Attendance | Continuous | Full history | Permanent |
| Vaccination | Continuous | Full history | Permanent |
| Health data | Continuous | Full history | Permanent |
| Behavior score | Continuous | Full history | Permanent |
| Student profile | Continuous | Full history | Permanent |

### **Purge Schedule**

- **Automatic**: Academic-year data older than 1 year automatically purged
- **Manual**: School can trigger purge at year boundary
- **Continuous data**: Never purged (kept for full history)
- **Audit logging**: Purge events logged for compliance

---

## Related Documents

- **[USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md)** - User personas and 95+ stories across phases
- **[TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)** - Flutter architecture, SQLite schema, API endpoints
- **[INDEX.md](INDEX.md)** - Master navigation and document relationships

---

**Document Version**: 1.0  
**Last Updated**: 2026-05-23  
**Status**: ✅ Final - Ready for Development

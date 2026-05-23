# Traceability Matrix - Complete Cross-References

> **Purpose**: Map all relationships between Requirements, User Stories, API Endpoints, and Database Tables  
> **Related Documents**: [INDEX.md](INDEX.md) | [REQUIREMENTS.md](REQUIREMENTS.md) | [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) | [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)  
> **Last Updated**: 2026-05-23

---

## Reference ID Conventions

| Type | Format | Example | Count |
|------|--------|---------|-------|
| **Requirement (Functional)** | FR-X.Y | FR-1.1, FR-2.3 | 50+ |
| **Requirement (Non-Functional)** | NFR-X.Y | NFR-1.1, NFR-5.2 | 30+ |
| **User Story** | US-XXX-PHASE | US-001-MVP, US-042-P2 | ~95 |
| **API Endpoint** | EP-X | EP-1 through EP-16 | 16 |
| **Database Table** | TB-X | TB-1 through TB-13 | 13 |

---

## Phase 1 (MVP) - Feature Traceability

### **Feature 1: Authentication & Guardian Management**

**Reference**: [REQUIREMENTS.md#1-authentication](REQUIREMENTS.md#1-authentication--guardian-management)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-1.1 through FR-1.9 | Guardian registration, login, OTP, child linking, device binding, revocation, password reset |
| | NFR-1.1 through NFR-1.3 | Performance, Security, Reliability |
| **User Stories** | US-001-MVP | Parent Registration |
| | US-002-MVP | Guardian Login |
| | US-003-MVP | Key Guardian Add First Child (With OTP) |
| | US-004-MVP | Additional Guardian Request Access |
| | US-005-MVP | Key Guardian Approve/Deny Access Request |
| | US-006-MVP | New Device Login Warning |
| | US-007-MVP | Revoke Additional Guardian Access |
| | US-008-MVP | Forgot Password |
| **API Endpoints** | EP-1 | Register Guardian (POST /api/v1/auth/register) |
| | EP-2 | Guardian Login (POST /api/v1/auth/login) |
| | EP-3 | Send OTP (POST /api/v1/auth/otp-send) |
| | EP-4 | Verify OTP & Link Child (POST /api/v1/auth/otp-verify) |
| | EP-5 | Register Device (POST /api/v1/auth/device-register) |
| **Database Tables** | TB-1 | guardian_profile |
| | TB-2 | devices |
| | TB-3 | guardian_children |

---

### **Feature 2: Sync & Offline Functionality**

**Reference**: [REQUIREMENTS.md#2-sync](REQUIREMENTS.md#2-sync--offline-functionality)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-2.1 through FR-2.7 | Data classification, initial sync, delta sync, conflict resolution, retry logic, FCM subscription, offline support |
| | NFR-2.1 through NFR-2.4 | Performance, Reliability, Bandwidth, Offline |
| **User Stories** | US-009-MVP | Initial Full Sync on First Child Link |
| | US-010-MVP | Background Delta Sync (10-Minute Cycle) |
| | US-011-MVP | View Data Offline (After Sync) |
| | US-012-MVP | Handle Sync Failures Gracefully |
| | US-013-MVP | Auto-Purge Old Academic Year Data |
| | US-014-MVP | Subscribe to FCM Notifications |
| **API Endpoints** | EP-6 | Initial Sync (POST /api/v1/sync/initial-login) |
| | EP-7 | Delta Sync (POST /api/v1/sync/delta) |
| **Database Tables** | TB-1 | guardian_profile (sync metadata) |
| | TB-2 | devices (last_sync tracking) |
| | TB-3 | guardian_children (sync per child) |
| | TB-4 | student_profile |
| | TB-5 | sync_metadata |
| | TB-6 | attendance |
| | TB-7 | homework |
| | TB-8 | exam |
| | TB-9 | exam_result |
| | TB-10 | class_activity |
| | TB-11 | class_routine |

---

### **Feature 3: Dashboard & Student Profile**

**Reference**: [REQUIREMENTS.md#3-dashboard](REQUIREMENTS.md#3-dashboard--student-profile)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-3.1 through FR-3.5 | Dashboard display, quick stats, child profile, photo display, school branding |
| | NFR-3.1, NFR-3.2 | Performance, Caching |
| **User Stories** | US-015-MVP | View Dashboard (Overview of Child's Status) |
| | US-016-MVP | View Child Profile & Photo |
| | US-017-MVP | View School Branding & Theme |
| | US-018-MVP | Switch Between Multiple Children |
| **API Endpoints** | EP-8 | Get Dashboard (GET /api/v1/dashboard/{child_id}) |
| | EP-9 | Get Guardian Children (GET /api/v1/guardian/children) |
| **Database Tables** | TB-4 | student_profile |
| | TB-12 | school_info |

---

### **Feature 4: School Information & Multi-Tenancy**

**Reference**: [REQUIREMENTS.md#4-school](REQUIREMENTS.md#4-school-information--multi-tenancy)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-4.1 through FR-4.4 | School search, directory, contacts, multi-school support |
| | NFR-4.1 | Data isolation |
| **User Stories** | US-019-MVP | Search & Add School |
| | US-020-MVP | View School Directory & Contacts |
| | US-021-MVP | Support Multiple Schools & Children |
| **API Endpoints** | EP-14 | Search Schools (GET /api/v1/schools/search) |
| | EP-15 | Get School Directory (GET /api/v1/schools/{school_id}/directory) |
| **Database Tables** | TB-12 | school_info |
| | TB-13 | school_contacts |

---

### **Feature 5: Attendance Intelligence**

**Reference**: [REQUIREMENTS.md#5-attendance](REQUIREMENTS.md#5-attendance-intelligence)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-5.1 through FR-5.4 | View attendance, monthly summary, trends, low attendance alerts |
| | NFR-5.1 | Real-time updates |
| **User Stories** | US-022-MVP | View Attendance Calendar (Month View) |
| | US-023-MVP | View Attendance Percentage & Trends |
| | US-024-MVP | Get Low Attendance Alerts |
| **API Endpoints** | EP-10 | Get Attendance (GET /api/v1/attendance/{child_id}) |
| **Database Tables** | TB-6 | attendance |

---

### **Feature 6: Class Routine**

**Reference**: [REQUIREMENTS.md#6-class-routine](REQUIREMENTS.md#6-class-routine)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-6.1, FR-6.2 | View class schedule, subject details |
| **User Stories** | US-025-MVP | View Class Routine/Schedule |
| **API Endpoints** | EP-6, EP-7 | (Included in Sync Endpoints) |
| **Database Tables** | TB-11 | class_routine |

---

### **Feature 7: Daily Class Activity**

**Reference**: [REQUIREMENTS.md#7-class-activity](REQUIREMENTS.md#7-daily-class-activity)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-7.1, FR-7.2 | View class activities, daily updates |
| **User Stories** | US-026-MVP | View Daily Class Activities |
| **API Endpoints** | EP-6, EP-7 | (Included in Sync Endpoints) |
| **Database Tables** | TB-10 | class_activity |

---

### **Feature 8: Homework & Assignment Tracker**

**Reference**: [REQUIREMENTS.md#8-homework](REQUIREMENTS.md#8-homework--assignment-tracker)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-8.1 through FR-8.4 | View homework, due dates, completion status, feedback |
| | NFR-8.1 | Real-time updates |
| **User Stories** | US-027-MVP | View Homework List (All & Pending) |
| | US-028-MVP | View Homework Details & Due Dates |
| | US-029-MVP | View Homework Feedback & Marks |
| **API Endpoints** | EP-11 | Get Homework List (GET /api/v1/homework/{child_id}) |
| **Database Tables** | TB-7 | homework |

---

### **Feature 9: Examination & Result Management**

**Reference**: [REQUIREMENTS.md#9-examination](REQUIREMENTS.md#9-examination--result-management)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-9.1 through FR-9.4 | View exam schedule, results, performance trends |
| | NFR-9.1 | Real-time result publishing |
| **User Stories** | US-030-MVP | View Exam Schedule |
| | US-031-MVP | View Exam Results & Grades |
| | US-032-MVP | View Performance Trends (Exams) |
| **API Endpoints** | EP-12 | Get Exam Schedule (GET /api/v1/exams/{child_id}) |
| | EP-13 | Get Exam Results (GET /api/v1/exam-results/{child_id}) |
| **Database Tables** | TB-8 | exam |
| | TB-9 | exam_result |

---

### **Feature 10: Notifications (Core Implementation)**

**Reference**: [REQUIREMENTS.md#10-notifications](REQUIREMENTS.md#10-notifications-core-implementation)

| Type | ID | Details |
|------|----|---------| 
| **Requirements** | FR-10.1 through FR-10.4 | Push notification delivery, in-app notifications, notification history, user preferences |
| | NFR-10.1 | 98% delivery rate |
| **User Stories** | US-033-MVP | Receive & View Push Notifications |
| | US-034-MVP | View Notification History |
| | US-035-MVP | Manage Notification Preferences |
| **API Endpoints** | EP-16 | Get Notifications (GET /api/v1/notifications) |
| | EP-6, EP-7 | (Notification updates in Sync) |
| **Database Tables** | TB-1, TB-2 | (FCM token in devices table) |

---

## API Endpoint Master List

| EP ID | Endpoint | HTTP Method | Path | Feature | Phase |
|-------|----------|-------------|------|---------|-------|
| **EP-1** | Register Guardian | POST | /api/v1/auth/register | Auth | MVP |
| **EP-2** | Guardian Login | POST | /api/v1/auth/login | Auth | MVP |
| **EP-3** | Send OTP | POST | /api/v1/auth/otp-send | Auth | MVP |
| **EP-4** | Verify OTP & Link Child | POST | /api/v1/auth/otp-verify | Auth | MVP |
| **EP-5** | Register Device | POST | /api/v1/auth/device-register | Auth | MVP |
| **EP-6** | Initial Sync | POST | /api/v1/sync/initial-login | Sync | MVP |
| **EP-7** | Delta Sync | POST | /api/v1/sync/delta | Sync | MVP |
| **EP-8** | Get Dashboard | GET | /api/v1/dashboard/{child_id} | Dashboard | MVP |
| **EP-9** | Get Guardian Children | GET | /api/v1/guardian/children | Dashboard | MVP |
| **EP-10** | Get Attendance | GET | /api/v1/attendance/{child_id} | Attendance | MVP |
| **EP-11** | Get Homework List | GET | /api/v1/homework/{child_id} | Homework | MVP |
| **EP-12** | Get Exam Schedule | GET | /api/v1/exams/{child_id} | Exam | MVP |
| **EP-13** | Get Exam Results | GET | /api/v1/exam-results/{child_id} | Exam | MVP |
| **EP-14** | Search Schools | GET | /api/v1/schools/search | School | MVP |
| **EP-15** | Get School Directory | GET | /api/v1/schools/{school_id}/directory | School | MVP |
| **EP-16** | Get Notifications | GET | /api/v1/notifications | Notifications | MVP |

---

## Database Table Master List

| TB ID | Table Name | Purpose | Feature |
|-------|------------|---------|---------|
| **TB-1** | guardian_profile | Store parent/guardian account info | Auth |
| **TB-2** | devices | Track registered devices per guardian | Auth |
| **TB-3** | guardian_children | Link guardians to children | Auth |
| **TB-4** | student_profile | Store student/child basic info | Dashboard |
| **TB-5** | sync_metadata | Track sync status per school | Sync |
| **TB-6** | attendance | Store daily attendance records | Attendance |
| **TB-7** | homework | Store homework assignments | Homework |
| **TB-8** | exam | Store exam schedules | Exam |
| **TB-9** | exam_result | Store exam results & grades | Exam |
| **TB-10** | class_activity | Store daily class activities | Class Activity |
| **TB-11** | class_routine | Store weekly class schedule | Class Routine |
| **TB-12** | school_info | Store school information | School |
| **TB-13** | school_contacts | Store school contact details | School |

---

## How to Use This Matrix

### **Trace a Requirement to Its Implementation**
1. Find the feature in this matrix (e.g., "Feature 1: Authentication")
2. See which FR/NFR codes apply (e.g., FR-1.1)
3. Check which User Stories implement it (e.g., US-001-MVP)
4. Check which API Endpoints satisfy it (e.g., EP-1)
5. Check which Database Tables store the data (e.g., TB-1)

**Example**: To implement FR-1.1 (Guardian registration):
- User Story: [US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp-parent-registration)
- API: [EP-1](TECH_ARCHITECTURE.md#1-register-guardian) - POST /api/v1/auth/register
- Database: [TB-1](TECH_ARCHITECTURE.md#1-guardian-profile-table) - guardian_profile table

### **Trace a User Story to Its Requirements**
1. Find the story in USER_STORIES_PERSONAS.md
2. Check the "Reference" field for FR codes and EP links
3. Jump to REQUIREMENTS.md for FR details
4. Jump to TECH_ARCHITECTURE.md for EP and TB details

### **Trace an API Endpoint to Its Purpose**
1. Find the endpoint in this matrix (e.g., EP-5)
2. Check which Feature it belongs to
3. Check which FR it satisfies
4. Check which TB it reads/writes
5. See TECH_ARCHITECTURE.md for full contract details

### **Trace a Database Table to Its Usage**
1. Find the table in this matrix (e.g., TB-6 = attendance)
2. Check which Feature uses it
3. Check which API Endpoints read/write it
4. Check which User Stories depend on it
5. See TECH_ARCHITECTURE.md for schema details

---

## Reference Links

All documents use consistent ID patterns for easy navigation:

- **Requirements**: `[FR-1.1](REQUIREMENTS.md#1-authentication)` = Functional Requirement 1.1
- **User Stories**: `[US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp-parent-registration)` = User Story 001 in MVP
- **API Endpoints**: `[EP-1](TECH_ARCHITECTURE.md#1-register-guardian)` = Endpoint 1
- **Database Tables**: `[TB-1](TECH_ARCHITECTURE.md#1-guardian-profile-table)` = Table 1

---

**Next Steps**: 
1. All four documents (REQUIREMENTS.md, USER_STORIES_PERSONAS.md, TECH_ARCHITECTURE.md) now include cross-references using these IDs
2. Use this matrix as a reference when developing features
3. When adding new features, update this matrix first, then add corresponding sections to other documents

# Phase 1 (MVP) Project Plan — Guardian Parent App

> **Related Documents**: [REQUIREMENTS.md](REQUIREMENTS.md) | [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) | [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)  
> **Last Updated**: 2026-05-23  
> **Phase**: 1 (MVP) — Core Features  
> **Status**: Planning

---

## Table of Contents

1. [Plan Overview](#plan-overview)
2. [Assumptions & Constraints](#assumptions--constraints)
3. [Sprint Schedule](#sprint-schedule)
   - [Sprint 1 (Weeks 1–2): Foundation & Authentication Basics](#sprint-1-weeks-12-foundation--authentication-basics)
   - [Sprint 2 (Weeks 3–4): Child Linking & Guardian Management](#sprint-2-weeks-34-child-linking--guardian-management)
   - [Sprint 3 (Weeks 5–6): Sync Engine](#sprint-3-weeks-56-sync-engine)
   - [Sprint 4 (Weeks 7–8): Dashboard & Multi-School](#sprint-4-weeks-78-dashboard--multi-school)
   - [Sprint 5 (Weeks 9–10): Academic Features](#sprint-5-weeks-910-academic-features)
   - [Sprint 6 (Weeks 11–12): Notifications & Quality Pass](#sprint-6-weeks-1112-notifications--quality-pass)
4. [Milestones](#milestones)
5. [Story Coverage Summary](#story-coverage-summary)
6. [Claude Code Usage Guide](#claude-code-usage-guide)
7. [Risks & Mitigations](#risks--mitigations)

---

## Plan Overview

| Item | Detail |
|------|--------|
| **Total Duration** | 12 weeks |
| **Working Hours/Week** | 15 hours |
| **Total Hours Available** | 180 hours |
| **Sprints** | 6 × 2-week sprints (30 hours each) |
| **Stories to Deliver** | 40 (all Phase 1) |
| **Development Approach** | Claude Code assisted Flutter development |
| **API Strategy** | Mock API layer throughout Phase 1 |
| **Testing** | Manual testing (device/emulator) |

**Hour allocation per sprint:**

| Category | Hours/Sprint |
|----------|-------------|
| Story development | ~23h |
| Manual testing & bug fixes | ~4h |
| Sprint review & planning | ~2h |
| Buffer | ~1h |

---

## Assumptions & Constraints

- **Existing codebase**: Review and align folder structure to the architecture defined in [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md) before Sprint 1 development begins
- **Mock API layer**: A single `MockApiService` will simulate all 16 API endpoints (EP-1 to EP-16) with realistic JSON responses. Real backend integration is Phase 2+ scope
- **Claude Code**: Used for all code generation — screens, providers, models, SQLite queries, and API service wiring
- **Bangla localization** (US-037): Set up in Sprint 1 and applied to every new screen as it is built — not a one-time task at the end
- **Mobile-responsive design** (US-038): Applied as a review pass in Sprint 4 across all screens built so far, then maintained in Sprints 5–6
- **No GDPR, no PCI-DSS** requirements in Phase 1 (bKash fee payment is Phase 2)
- **FCM setup** required before Sprint 6 notifications work — create Firebase project and add `google-services.json` / `GoogleService-Info.plist` before Week 11

---

## Sprint Schedule

### Sprint 1 (Weeks 1–2): Foundation & Authentication Basics

**Goal**: Working app shell with registration, login, school discovery, and Bangla localization wired up.

**Hours**: 30h

#### Setup Tasks (not user stories)

| Task | Hours |
|------|-------|
| Review existing codebase; align to [TECH_ARCHITECTURE.md project structure](TECH_ARCHITECTURE.md#project-structure) | 3h |
| Set up `MockApiService` — EP-1 to EP-5 mock responses | 2h |
| Set up SQLite provider + core schemas: TB-1 (guardian_profile), TB-2 (devices), TB-3 (guardian_children) | 3h |
| Set up `go_router` navigation + Provider/Riverpod state management | 1h |

#### Stories

| Story | Description | Priority | Est. Hours |
|-------|-------------|----------|-----------|
| [US-037-MVP](USER_STORIES_PERSONAS.md#us-037-mvp-bangla-language-support) | Bangla localization — set up `intl`, ARB files, language toggle (applied to every screen going forward) | MUST | 5h |
| [US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp-parent-registration) | Registration screen — email/password validation, API call to EP-1 | MUST | 2h |
| [US-002-MVP](USER_STORIES_PERSONAS.md#us-002-mvp-guardian-login) | Login screen — JWT storage, lockout handling, EP-2 | MUST | 2h |
| [US-008-MVP](USER_STORIES_PERSONAS.md#us-008-mvp-forgot-password) | Forgot password screen — email reset flow | SHOULD | 2h |
| [US-017-MVP](USER_STORIES_PERSONAS.md#us-017-mvp-school-selectiondiscovery) | School search/discovery screen — EP-14 mock | MUST | 2h |
| [US-018-MVP](USER_STORIES_PERSONAS.md#us-018-mvp-school-branding-in-app) | School branding — apply theme colors and logo from sync | SHOULD | 2h |

**Testing focus**: Register → Login → School selection flow end-to-end on emulator

**Sprint 1 Deliverable**: App launches, user can register, log in, and select a school. Bangla/English toggle works.

---

### Sprint 2 (Weeks 3–4): Child Linking & Guardian Management

**Goal**: Both guardian types can link a child. Key Guardian can approve/revoke Additional Guardian access.

**Hours**: 30h

#### Setup Tasks

| Task | Hours |
|------|-------|
| Set up remaining SQLite schemas: TB-4 through TB-13 (student_profile, sync_metadata, attendance, homework, exam, exam_result, class_activity, class_routine, school_info, school_contacts) | 4h |
| Add EP-3, EP-4, EP-5, EP-9 mock responses to `MockApiService` | 1h |

#### Stories

| Story | Description | Priority | Est. Hours |
|-------|-------------|----------|-----------|
| [US-003-MVP](USER_STORIES_PERSONAS.md#us-003-mvp-key-guardian-add-first-child-with-otp) | Key Guardian: Student ID + DOB → OTP screen → child linked → initial sync triggered | MUST | 6h |
| [US-004-MVP](USER_STORIES_PERSONAS.md#us-004-mvp-additional-guardian-request-access) | Additional Guardian: request access flow, 48h expiry, awaiting approval state | MUST | 4h |
| [US-005-MVP](USER_STORIES_PERSONAS.md#us-005-mvp-key-guardian-approvedeny-access-request) | Key Guardian: pending requests list, approve/deny buttons, sync trigger | MUST | 4h |
| [US-006-MVP](USER_STORIES_PERSONAS.md#us-006-mvp-new-device-login-warning) | New device login warning dialog — old device lock logic | MUST | 2h |
| [US-007-MVP](USER_STORIES_PERSONAS.md#us-007-mvp-revoke-additional-guardian-access) | Revoke additional guardian — confirmation, revocation flag | MUST | 2h |
| [US-019-MVP](USER_STORIES_PERSONAS.md#us-019-mvp-view-school-contacts) | School contacts directory — tap-to-call, tap-to-email, EP-15 mock | SHOULD | 2h |

**Testing focus**: Key Guardian OTP flow, Additional Guardian request → Key Guardian approve → sync triggered, revocation flow

**Sprint 2 Deliverable**: Both guardian flows complete. Access approval and revocation work correctly.

---

### Sprint 3 (Weeks 5–6): Sync Engine

**Goal**: Full initial sync and 10-minute background delta sync working against mock API. Offline mode functional.

**Hours**: 30h

> This is the most technically complex sprint. The sync engine underlies every feature — allocate maximum focus here.

#### Setup Tasks

| Task | Hours |
|------|-------|
| Add EP-6 (initial sync) and EP-7 (delta sync) mock responses — realistic JSON payloads with all data types | 2h |

#### Stories

| Story | Description | Priority | Est. Hours |
|-------|-------------|----------|-----------|
| [US-009-MVP](USER_STORIES_PERSONAS.md#us-009-mvp-initial-full-sync-on-first-login) | Initial full sync: blocking progress bar (3 steps), all data downloaded to SQLite, school branding applied, FCM topics subscribed, 3-failure fallback | MUST | 10h |
| [US-010-MVP](USER_STORIES_PERSONAS.md#us-010-mvp-background-10-minute-delta-sync) | Background 10-min delta sync: timer, per-school requests, merge changed records into SQLite, update sync_metadata, "Last synced: just now" UI | MUST | 10h |
| [US-011-MVP](USER_STORIES_PERSONAS.md#us-011-mvp-offline-data-access) | Offline mode: "Offline" banner, data staleness display (<24h / 24-48h warning / >7 days alert), read-only enforcement | MUST | 3h |
| [US-012-MVP](USER_STORIES_PERSONAS.md#us-012-mvp-automatic-sync-on-connection-return) | Auto-sync on reconnect: `connectivity_plus` listener triggers sync immediately | SHOULD | 2h |

**Testing focus**: Simulate offline mode, test all 3 staleness thresholds, verify delta merge doesn't duplicate records, confirm sync_metadata updates correctly

**Sprint 3 Deliverable**: App fully syncs data from mock API, works offline, and auto-recovers when connection returns.

---

### Sprint 4 (Weeks 7–8): Dashboard & Multi-School

**Goal**: Dashboard screen live from local SQLite. Multi-child and multi-school switching working. All existing screens responsive.

**Hours**: 30h

#### Stories

| Story | Description | Priority | Est. Hours |
|-------|-------------|----------|-----------|
| [US-013-MVP](USER_STORIES_PERSONAS.md#us-013-mvp-revocation-detection-via-sync) | Revocation detection: parse sync `revocation_status` flag → clear data → force logout → show in-app message | MUST | 3h |
| [US-014-MVP](USER_STORIES_PERSONAS.md#us-014-mvp-view-child-dashboard) | Dashboard: student name/photo/class, school branding, quick stats (attendance %, next homework, next exam, low-attendance alert badge), last synced timestamp | MUST | 6h |
| [US-015-MVP](USER_STORIES_PERSONAS.md#us-015-mvp-multi-child-profile-switching) | Multi-child dropdown/carousel — instant switch (<500ms), remember last selected | SHOULD | 2h |
| [US-016-MVP](USER_STORIES_PERSONAS.md#us-016-mvp-view-detailed-student-profile) | Student profile: all fields read-only, guardian info, emergency contacts | SHOULD | 1h |
| [US-033-MVP](USER_STORIES_PERSONAS.md#us-033-mvp-add-child-from-different-school) | Add child from different school — separate school_id, independent sync cycle | SHOULD | 4h |
| [US-034-MVP](USER_STORIES_PERSONAS.md#us-034-mvp-view-school-context-properly) | School context display — school name/branding per child, per-school last synced time | SHOULD | 2h |
| [US-038-MVP](USER_STORIES_PERSONAS.md#us-038-mvp-mobile-responsive-design) | Mobile-responsive review pass — all screens built so far tested on 4.5" and 6.5" sizes, 48×48dp touch targets enforced, no horizontal scroll | MUST | 8h |

**Testing focus**: Dashboard accuracy (stats match SQLite data), multi-child switching, screen sizes on emulator (small and large phones)

**Sprint 4 Deliverable**: Dashboard is the app's home screen. Multi-child and multi-school work. All screens are mobile-responsive.

---

### Sprint 5 (Weeks 9–10): Academic Features

**Goal**: Attendance, homework, exams, class activity, and class routine all displaying from local SQLite.

**Hours**: 30h

#### Stories

| Story | Description | Priority | Est. Hours |
|-------|-------------|----------|-----------|
| [US-020-MVP](USER_STORIES_PERSONAS.md#us-020-mvp-view-daily-attendance-calendar) | Attendance calendar: 30-day color-coded calendar (Present/Absent/Leave/Half-day), tap date for remarks, monthly % | MUST | 3h |
| [US-021-MVP](USER_STORIES_PERSONAS.md#us-021-mvp-low-attendance-alert) | Low attendance alert: <75% triggers in-app badge on dashboard + push notification to Key Guardian | MUST | 2h |
| [US-022-MVP](USER_STORIES_PERSONAS.md#us-022-mvp-attendance-trend-analysis) | Attendance analytics: weekly bar chart (last 4 weeks), monthly line chart (last 6 months), trend vs. previous month | SHOULD | 4h |
| [US-023-MVP](USER_STORIES_PERSONAS.md#us-023-mvp-view-homework-list) | Homework list: all statuses, marks/grade/feedback display, filter by subject, sort by due date, mark as viewed | MUST | 5h |
| [US-024-MVP](USER_STORIES_PERSONAS.md#us-024-mvp-homework-due-reminder) | Homework due reminder: "Due tomorrow" label in-app (push reminder wired in Sprint 6) | SHOULD | 1h |
| [US-025-MVP](USER_STORIES_PERSONAS.md#us-025-mvp-view-exam-schedule) | Exam schedule: chronological list, calendar highlighting, exam type/duration/instructions | MUST | 2h |
| [US-026-MVP](USER_STORIES_PERSONAS.md#us-026-mvp-view-exam-results) | Exam results: marks/total, grade, percentile, class average, subject breakdown | MUST | 3h |
| [US-027-MVP](USER_STORIES_PERSONAS.md#us-027-mvp-exam-performance-analytics) | Exam analytics: subject trend line graph, radar chart across subjects, performance band | SHOULD | 3h |
| [US-028-MVP](USER_STORIES_PERSONAS.md#us-028-mvp-view-daily-class-updates) | Daily class activity feed: last 7 days, card view, filter by subject, load older | SHOULD | 3h |
| [US-029-MVP](USER_STORIES_PERSONAS.md#us-029-mvp-view-class-timetable) | Class routine timetable: weekly view, today highlighted, tap teacher for contact | SHOULD | 2h |

**Testing focus**: All data renders correctly from SQLite mock data, filter/sort functions, charts render within 1 second, offline access for all screens

**Sprint 5 Deliverable**: All academic data features complete. App usable as a read-only academic tracker.

---

### Sprint 6 (Weeks 11–12): Notifications & Quality Pass

**Goal**: Push notifications live for Key Guardian. All edge cases handled. App polished and ready for Phase 1 sign-off.

**Hours**: 30h

> **Pre-condition**: Firebase project must be created and `google-services.json` / `GoogleService-Info.plist` added to the project before this sprint starts.

#### Stories

| Story | Description | Priority | Est. Hours |
|-------|-------------|----------|-----------|
| [US-030-MVP](USER_STORIES_PERSONAS.md#us-030-mvp-push-notifications-for-key-events) | FCM push notifications to Key Guardian: homework due, exam announced, low attendance, behavioral concerns, assignment evaluated. Tap notification opens correct screen | MUST | 6h |
| [US-031-MVP](USER_STORIES_PERSONAS.md#us-031-mvp-notification-preferences) | Notification preferences: enable/disable per category, quiet hours (9PM–8AM), frequency control | SHOULD | 2h |
| [US-032-MVP](USER_STORIES_PERSONAS.md#us-032-mvp-notification-history) | Notification history: reverse chronological list, mark read/unread, search | NICE | 1h |
| [US-035-MVP](USER_STORIES_PERSONAS.md#us-035-mvp-app-loads-quickly) | Performance: startup <3s, dashboard load <1s, page transitions <500ms — profile with Flutter DevTools and fix bottlenecks | SHOULD | 2h |
| [US-036-MVP](USER_STORIES_PERSONAS.md#us-036-mvp-graceful-error-handling) | Error handling: user-friendly messages for all error states, retry buttons, sync failure fallback | SHOULD | 3h |
| [US-039-MVP](USER_STORIES_PERSONAS.md#us-039-mvp-low-bandwidth-optimization) | Bandwidth optimisation: image compression, lazy loading, verify delta sync payload sizes | SHOULD | 2h |
| [US-040-MVP](USER_STORIES_PERSONAS.md#us-040-mvp-battery-efficient) | Battery efficiency: verify background sync uses <3% per cycle, no excessive wake-locks | NICE | 1h |

#### Quality Pass (not user stories)

| Task | Hours |
|------|-------|
| Full end-to-end manual test: Key Guardian flow (register → link child → sync → all features) | 3h |
| Full end-to-end manual test: Additional Guardian flow (request → approval → data access → revocation) | 2h |
| Test on Android physical device (minimum 2GB RAM) | 2h |
| Test on iOS (if available) | 1h |
| Bug fixes from testing | 5h |

**Sprint 6 Deliverable**: Phase 1 MVP complete. All 40 stories delivered. App stable on Android. Ready for user acceptance testing.

---

## Milestones

| Milestone | Week | Criteria |
|-----------|------|----------|
| **M1: Auth Complete** | End of Week 4 | Both guardian types can register, log in, link children, and manage access |
| **M2: Sync Engine Complete** | End of Week 6 | Initial sync and delta sync work; offline mode functional |
| **M3: Dashboard Live** | End of Week 8 | Dashboard shows real data from SQLite; multi-child/school works; all screens responsive |
| **M4: Academic Features Complete** | End of Week 10 | Attendance, homework, exams, class activity, class routine all working |
| **M5: Phase 1 MVP Complete** | End of Week 12 | All 40 stories delivered, notifications live, quality pass done |

---

## Story Coverage Summary

| Sprint | Weeks | Stories | MUST | SHOULD | NICE | Hours |
|--------|-------|---------|------|--------|------|-------|
| Sprint 1 | 1–2 | US-001, 002, 008, 017, 018, 037 + Setup | 3 | 3 | 0 | 30h |
| Sprint 2 | 3–4 | US-003, 004, 005, 006, 007, 019 + DB Setup | 5 | 1 | 0 | 30h |
| Sprint 3 | 5–6 | US-009, 010, 011, 012 | 3 | 1 | 0 | 30h |
| Sprint 4 | 7–8 | US-013, 014, 015, 016, 033, 034, 038 | 3 | 4 | 0 | 30h |
| Sprint 5 | 9–10 | US-020, 021, 022, 023, 024, 025, 026, 027, 028, 029 | 4 | 6 | 0 | 30h |
| Sprint 6 | 11–12 | US-030, 031, 032, 035, 036, 039, 040 + QA | 1 | 4 | 2 | 30h |
| **Total** | **12 weeks** | **40 stories** | **19** | **19** | **2** | **180h** |

> **Note**: US-013 (Revocation detection) maps to MUST in requirements; US-033/034 map to SHOULD. Adjusted counts reflect actual story priorities.

---

## Claude Code Usage Guide

How to use Claude Code most effectively during each sprint:

### What Claude Code does best
- Generating screen scaffolding from a description or wireframe
- Writing SQLite CRUD queries from a table schema
- Building Provider/Riverpod state classes from a data model
- Wiring mock API responses to match the EP-x contract in TECH_ARCHITECTURE.md
- Writing form validation logic and error message handling
- Generating `freezed` model classes from JSON structures

### Effective prompting pattern
When starting a new story, provide Claude Code with:
1. The user story ID and acceptance criteria from [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md)
2. The relevant FR-x requirements from [REQUIREMENTS.md](REQUIREMENTS.md)
3. The API endpoint contract (EP-x) from [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)
4. The SQLite table schema (TB-x) from [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)

**Example**: *"Build the login screen for US-002-MVP. Requirements are in FR-1.2. The API contract is EP-2. Store the JWT in TB-1. Show account lockout after 3 failed attempts."*

### Per-sprint Claude Code focus
| Sprint | Primary Claude Code tasks |
|--------|--------------------------|
| 1 | Generate project structure, auth screens, ARB localization files |
| 2 | OTP screen, approval flows, SQLite schema creation scripts |
| 3 | SyncService class, SQLite merge logic, connectivity listener |
| 4 | Dashboard widgets, Provider state classes, responsive layout pass |
| 5 | Chart widgets, list/filter/sort screens, SQLite read queries |
| 6 | FCM setup, notification routing, error boundary widgets |

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Sync engine (Sprint 3) takes longer than estimated | High | High | Sprint 3 is protected — only 4 stories. If US-009 overruns, move US-012 to Sprint 4 |
| FCM setup blocked (no Firebase project ready) | Medium | Medium | Create Firebase project before Week 11 starts — do not wait until Sprint 6 |
| Existing codebase conflicts with architecture | Medium | Medium | Spend first 3 hours of Sprint 1 auditing the codebase; refactor early, not late |
| Mobile-responsive pass (US-038) underestimated | Medium | Low | 8h allocated in Sprint 4; if more needed, trim US-039/040 (SHOULD/NICE) |
| Sprint 5 has 10 stories — may overrun | Medium | Medium | All Sprint 5 stories are short (1–5h each). If needed, defer US-027 (exam analytics, SHOULD) to Sprint 6 |
| Manual testing finds critical bugs late | Low | High | Test each screen immediately after building — don't batch all testing to Sprint 6 |

---

**Document Version**: 1.0  
**Phase**: 1 (MVP)  
**Last Updated**: 2026-05-23  
**Status**: Ready for Sprint 1

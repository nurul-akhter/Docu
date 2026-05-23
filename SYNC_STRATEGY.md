# Mobile App Sync Strategy

## Overview

Guardian mobile app uses SQLite local database with periodic delta sync to school backend. All data is stored locally and works offline. Background syncs keep data fresh when online.

**Key Principles**:

- **Offline-first**: Instant local reads, works without internet
- **Delta sync**: Only changed data is synced (10-minute background cycles)
- **Data classification**: Academic year data (expires yearly) + continuous data (permanent history)
- **School-based**: Syncs grouped by school; guardians with multiple schools get separate sync cycles per school
- **Device-bound**: Each device maintains its own sync state; new device triggers fresh full sync

---

## 1. Architecture Layers

```
┌─────────────────────────────────────┐
│     Guardian Mobile App             │
│  (iOS/Android with SQLite)          │
│  - Instant local reads              │
│  - Background 10-min sync per school│
│  - Offline-first experience         │
│  - Device-specific sync metadata    │
└─────────────────────────────────────┘
           ↓ (per-school sync)
┌─────────────────────────────────────┐
│     REST API (School Backend)       │
│  - Per-school + student change log  │
│  - Delta change detection           │
│  - Guardian data authorization      │
│  - Device revocation detection      │
└─────────────────────────────────────┘
           ↓ (read/write)
┌─────────────────────────────────────┐
│  PostgreSQL (School Database)       │
│  - School + student change log      │
│  - All school data                  │
│  - Multi-tenancy with RLS           │
└─────────────────────────────────────┘
```

---

## 2. Data Classification

All data falls into one of two categories:

### Academic Year Data

- **Lifecycle**: Current academic year only (e.g., 2025-2026)
- **Examples**: Exam schedule, exam results, class activities, current homework, class routine for current year
- **Purge**: Automatically purged at academic year boundary
- **Retention**: Current year + previous year max (for reference)
- **Sync**: Included in initial sync; ongoing changes synced every 10 minutes

### Continuous Data

- **Lifecycle**: From enrollment to present (permanent)
- **Examples**: Vaccination records, health & growth measurements, behavior & discipline history, student profile info
- **Purge**: Never purged
- **Retention**: Full historical data always available
- **Sync**: Included in initial sync; ongoing changes synced every 10 minutes

### Classification Table

Backend maintains a table mapping data type → classification:

```
data_type          | classification  | description
─────────────────────────────────────────────────
exam               | academic_year   | Exam schedule & results
homework           | academic_year   | Class assignments
class_activity     | academic_year   | In-class activities, projects
class_routine      | academic_year   | Current year schedule
vaccination        | continuous      | Full history
health_data        | continuous      | Height, weight over time
behavior_score     | continuous      | Discipline & behavior history
student_profile    | continuous      | Name, DOB, photo, basic info
attendance         | academic_year   | Attendance records
```

---

## 3. Initial Sync (First Login / New Device)

**Trigger**:

- First time guardian logs in after registration (Flow 4 / Flow 5)
- Guardian logs in from a new device (Flow 3)

### Sync Request

```json
{
  "school_id": "uuid",
  "guardian_id": "uuid",
  "child_ids": ["uuid1", "uuid2"],
  "sync_type": "initial",
  "device_id": "device-uuid",
  "current_datetime": "2026-05-21T10:30:00Z"
}
```

### Backend Processing

For each school, backend returns:

```json
{
  "sync_timestamp": "2026-05-21T10:30:00Z",
  "sync_type": "initial",
  "school_info": {
    "school_id": "uuid",
    "name": "Spring Valley School",
    "logo_url": "...",
    "theme": {...}
  },
  "children": [
    {
      "child_id": "uuid1",
      "student_profile": {...},
      "academic_year_data": {
        "current_academic_year": "2025-2026",
        "exam": [...],
        "homework": [...],
        "class_activity": [...],
        "class_routine": [...]
      },
      "continuous_data": {
        "vaccination": [...],
        "health_data": [...],
        "behavior_score": [...],
        "attendance": [...]  // full history
      }
    },
    {
      "child_id": "uuid2",
      ...
    }
  ]
}
```

### App Action Sequence

```
1. Capture device info:
   - device_id (generated and stored locally)
   - fcm_token (Firebase Cloud Messaging)
   - brand, model, os_version

2. Register device:
   POST /api/auth/device-register

3. For each school with children:
   - Create local SQLite DB (if first login)
   - Request initial sync
   - Merge all data (academic year + continuous)
   - Store sync_metadata.last_sync_datetime = sync_timestamp

4. Subscribe to FCM topics:
   - "school_{school_id}"
   - "child_{child_id}" (for each child)
   - "guardian_{guardian_id}"

5. Show "Setup complete ✓"
```

**Blocking Behavior**: Initial sync must complete before showing dashboard. If sync fails 3 times, allow offline mode with cached data.

---

## 4. Background 10-Minute Sync Cycle

**Runs automatically every 10 minutes** (only when online).

### Step 1: Check Network & Device Status

```
IF device has internet connection AND device_id is still active:
  Proceed to Step 2
ELSE:
  Skip sync, wait for next 10-minute cycle
```

### Step 2: Prepare Sync Request (Per School)

For each school the guardian has children in:

```json
{
  "school_id": "uuid",
  "guardian_id": "uuid",
  "child_ids": ["uuid1", "uuid2"],
  "sync_type": "delta",
  "device_id": "device-uuid",
  "last_sync_datetime": "2026-05-21T10:20:00Z", // from sync_metadata
  "current_datetime": "2026-05-21T10:30:00Z"
}
```

**Note**: One request per school. If guardian has children in 3 schools, send 3 requests (parallel or sequential).

### Step 3: Backend Change Detection

Backend maintains per-student, per-school change log:

```sql
CREATE TABLE data_change_log (
  school_id UUID NOT NULL,
  student_id UUID NOT NULL,
  data_type VARCHAR NOT NULL,
  last_updated_at TIMESTAMP,
  PRIMARY KEY (school_id, student_id, data_type)
);
```

**For each student in request**:

```
FOR each data_type (exam, homework, vaccination, health_data, etc):
  IF data_change_log[school_id, student_id, data_type].last_updated_at > last_sync_datetime:
    Mark this data_type as "has changes"
    Query records changed in range [last_sync_datetime, current_datetime]
    Include in response
  ELSE:
    Skip this data_type (no changes)
```

### Step 4: API Response (Only Changed Data)

```json
{
  "sync_timestamp": "2026-05-21T10:30:00Z",
  "sync_type": "delta",
  "school_id": "uuid",
  "revocation_status": {
    "is_revoked": false,
    "reason": null
  },
  "children": [
    {
      "child_id": "uuid1",
      "changes": {
        "homework": [
          {
            "id": "uuid",
            "subject": "Math",
            "title": "Chapter 5 Exercises",
            "due_date": "2026-05-22",
            "status": "assigned",
            "updated_at": "2026-05-21T10:28:00Z"
          }
        ],
        "health_data": [
          {
            "id": "uuid",
            "measurement_date": "2026-05-21",
            "height_cm": 165,
            "weight_kg": 58,
            "updated_at": "2026-05-21T10:25:00Z"
          }
        ]
      },
      "void_records": {
        "homework": [
          {
            "id": "uuid-of-homework",
            "status": "void"
          }
        ]
      }
    }
  ]
}
```

**Efficiency**: Most 10-minute syncs return only 10-20 KB (gzip compressed). If no changes across all students, response is minimal.

### Step 5: Local Database Merge

```
FOR each child in response:
  FOR each data_type in response.changes:
    FOR each record:
      IF record.id exists locally:
        UPDATE local record
      ELSE:
        INSERT new record

  FOR each data_type in response.void_records:
    FOR each record:
      UPDATE local record SET status = 'void'
      (Do NOT delete records; keep full history)

UPDATE sync_metadata:
  last_sync_datetime = response.sync_timestamp
  last_sync_status = 'success'
  last_sync_error = NULL
  last_sync_time = NOW()

Notify UI: Data updated, show "Last synced: just now"
```

---

## 5. Revocation & Device Handling

### Revocation Detection (Additional Guardian)

**Scenario**: Key guardian revokes an additional guardian's access.

**Trigger**: Immediate sync via push notification + 10-minute fallback.

**Backend Action**:

1. Mark guardian status → 'revoked'
2. Send FCM push: "Your access has been revoked"
3. Set `data_change_log` for all children to trigger sync

**App Action on Next Sync**:

```
1. Receive sync response with revocation_status.is_revoked = true
2. Clear local SQLite data
3. Clear JWT token
4. Force immediate logout
5. Show: "Your access has been revoked. Contact the key guardian."
6. Redirect to login screen
```

### Immediate Sync Trigger (via FCM)

When revocation happens, backend sends push notification:

```json
{
  "type": "revocation_notice",
  "message": "Your access has been revoked",
  "action": "sync"
}
```

**App receives notification**:

- If app is in foreground: trigger sync immediately
- If app is in background: sync on next wake-up (not later than 10-minute cycle)

### New Device Login

**Scenario**: Guardian logs in from a new device (Flow 3).

```
1. Backend detects device_id is new
2. Shows warning: "Logging in here will lock your previous device"
3. On Continue:
   - Old device_id is marked as "locked" (cannot sync)
   - New device_id is registered
   - Full initial sync triggered on new device
   - Old device: next sync fails, forces logout
```

**Old Device Behavior**:

- Can continue displaying cached data offline
- Next sync attempt detects it's locked
- Shows: "You logged in from another device. Session ended."
- Redirected to login screen

---

## 6. Offline Experience

### When Offline

```
- App displays cached local data instantly
- All read features work (view homework, exam schedule, health records, etc.)
- No edits allowed (prevent sync conflicts)
- Show indicator: "Offline - last synced 2 hours ago"
```

### When Connection Returns

```
- Automatic sync triggered immediately
- Background process, no interruption
- UI updates with fresh data
- If revocation detected: immediate logout
```

### Stale Data Handling

```
- If offline < 24 hours: show timestamp "Last updated: 2 hours ago"
- If offline 24-48 hours: show warning banner "Data may be out of date"
- If offline > 7 days: show alert "Last updated X days ago"
- Never show incorrect data; always flag staleness
```

---

## 7. Sync Metadata (App Side)

**Stored per school** (one entry per school the guardian has children in):

```sql
CREATE TABLE sync_metadata (
  school_id TEXT PRIMARY KEY,
  last_sync_datetime TEXT,
  last_sync_status TEXT,  -- success, failed, revoked, locked
  last_sync_error TEXT,
  last_sync_time TEXT,
  device_locked BOOLEAN,  -- set to true if device is locked by new login
  created_at TEXT
);
```

**Usage**:

- Read on every 10-minute cycle
- Update after each sync
- Clear on logout or revocation

---

## 8. Error Handling & Retry Logic

### Network Timeout

```
Show: "Last sync failed - retrying in 10 minutes"
Retry: At next 10-minute cycle
Action: No immediate retry (battery friendly)
```

### Server Error (5xx)

```
Log error with timestamp
Increment retry counter in sync_metadata
Retry: At next 10-minute cycle
Alert: If consecutive 5 failures, show "Data sync failed - contact support"
```

### Authentication Failure (401)

```
Reason: JWT expired or revoked
Action: Force re-login immediately
Show: "Your session expired. Please log in again."
Clear: JWT token, device_id, local data
```

### Authorization Failure (403)

```
Reason: Guardian no longer has access to this child
Action: Mark child as "access_denied"
Show: "You no longer have access to [Child Name]"
Local: Keep existing cached data (read-only mode)
```

### Validation Error (400)

```
Log error details
Skip this sync cycle
Alert: "Data sync failed - contact support"
Include: Error details for debugging
```

### Revocation Detection (Sync Response)

```
IF sync_response.revocation_status.is_revoked == true:
  Clear all local data
  Clear JWT token
  Force logout immediately
  Show: "Your access has been revoked. Contact the key guardian."
```

---

## 9. Bandwidth Optimization

### First Login (Initial Sync)

**Size**: 500 KB - 2 MB (varies by student history)

- Student profile: ~5 KB
- Academic year data (current): ~50-200 KB
- Continuous data (full history): ~400-1800 KB
- Gzip compressed: ~50% reduction

### Regular 10-Minute Sync

**Size**: 10-50 KB typical

- Most cycles: 10-20 KB (few changes)
- Busy periods: 30-50 KB (homework, activities, attendance updates)
- Change log prevents querying unchanged data
- Gzip compressed: ~50% reduction

### Monthly Usage (Estimated)

```
Background syncs: 4,320 cycles/month (1 per 10 min, 8 hours/day average usage)
Average per sync: 20 KB
Total: ~86 MB/month (uncompressed)
       ~43 MB/month (gzip compressed)
Plus initial syncs: ~1 MB per new device
```

**Acceptable** for mobile data plans in Bangladesh (typical plans: 1-5 GB/month).

---

## 10. Success Metrics

- ✅ Offline-first experience (works without internet)
- ✅ Background sync (non-blocking, battery-friendly, 10-min interval)
- ✅ Bandwidth efficient (delta sync, change log optimization)
- ✅ Data freshness: worst case 10 minutes (plus immediate sync on revocation)
- ✅ No data loss on sync failures
- ✅ Handles device switching seamlessly (lock + fresh sync)
- ✅ Revocation detected within 10 minutes (immediate push fallback)
- ✅ <500 MB local storage per guardian (academic year + continuous data)
- ✅ <3% battery drain from sync (10-min interval, gzip compression)
- ✅ Sub-second local data access (SQLite performance)

---

## 11. Sync Scenarios & Flows

### Scenario 1: New Guardian Registration

```
1. Guardian registers (Flow 1)
2. Adds first child (Flow 4: Key Guardian or Flow 5: Additional Guardian)
3. OTP verified (key) or approval received (additional)
4. Device registered
5. Initial sync triggered (blocking)
6. Sync metadata created for school
7. Dashboard loads with data
8. Background sync starts (next 10-min cycle)
```

### Scenario 2: Returning Guardian Login

```
1. Guardian logs in (Flow 2)
2. Backend validates credentials
3. Device is recognized (same device_id)
4. Dashboard loads with cached data immediately
5. Background sync triggers automatically
6. Sync response includes any new data
7. UI updates "Last synced: just now"
```

### Scenario 3: New Device Login

```
1. Guardian logs in from new device (Flow 3)
2. Backend detects new device_id
3. Warning shown: "Logging in will lock previous device"
4. On Continue:
   - Device registered with new device_id
   - Old device_id marked as locked
   - Initial sync triggered
   - All data downloaded fresh
5. Old device: next sync attempt fails, forces logout
6. New device: background sync continues normally
```

### Scenario 4: Guardian Revoked (Additional Guardian)

```
1. Key guardian revokes access (Flow 7)
2. Backend marks guardian as 'revoked'
3. FCM push sent immediately: "Your access has been revoked"
4. Additional guardian's device receives notification
   a. If in foreground: immediate sync triggered
   b. If in background: syncs on next wake-up
5. Sync response includes revocation_status.is_revoked = true
6. App clears data, logs out, shows message
7. Redirected to login screen
```

### Scenario 5: Multiple Children, Same School

```
1. Guardian has Child1 and Child2 at School A
2. Background sync (10 minutes):
   - Single request sent to School A API
   - Includes: school_id, child_ids = [uuid1, uuid2]
   - Response includes changes for both children
   - Sync metadata updated for School A (one timestamp)
3. Backend tracking (per student):
   - Separate change log for Child1
   - Separate change log for Child2
4. App local storage:
   - Tables for Child1 data
   - Tables for Child2 data
   - One sync_metadata entry for School A
```

### Scenario 6: Multiple Children, Different Schools

```
1. Guardian has Child1 at School A, Child2 at School B
2. Background sync (10 minutes):
   - Request 1 to School A API (for Child1)
   - Request 2 to School B API (for Child2)
   - Parallel or sequential
3. Responses merged into local DB
4. Sync metadata updated for both schools
5. UI shows: "Last synced at School A: 2 min ago, School B: 5 min ago"
```

---

## 12. Transition from Previous Year to New Academic Year

**Trigger**: Automatic at academic year boundary (configurable by school).

**Backend Action**:

```
1. Purge academic_year data older than 1 year
2. Retain current year + previous year academic data
3. Keep all continuous data (vaccination, health, behavior history)
4. Update data_change_log to mark all students
```

**App Action** (on next sync after boundary):

```
1. Sync response includes purged data notifications
2. Delete purged records locally
3. Update UI to show current academic year
4. Historical data still available in continuous section
```

---

## Notes & Future Considerations

- **Data Model**: Database structure defined separately (refer to DATABASE_DESIGN.md)
- **Immediate Sync**: Triggered via FCM push on critical events (revocation, critical announcements)
- **Classification Table**: Backend maintains centralized mapping of data type → classification (academic year vs continuous)
- **RLS (Row-Level Security)**: PostgreSQL enforces per-school, per-guardian data access
- **Request Signing**: Optional HTTPS-only, with request signatures for extra security (Phase 2)

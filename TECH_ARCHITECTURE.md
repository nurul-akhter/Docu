# Technical Architecture Document

> **Related Documents**: [INDEX.md](INDEX.md) | [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) | [REQUIREMENTS.md](REQUIREMENTS.md) | [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md)  
> **Last Updated**: 2026-05-23  
> **Platform**: Flutter (iOS 13+, Android 8+)  
> **Local Storage**: SQLite  
> **API Format**: REST JSON  
> **Status**: All APIs (EP-1 to EP-16) and Tables (TB-1 to TB-13) with cross-references

---

## How to Use This Document

**All API endpoints and database tables include explicit cross-references**:

- **EP-X (API Endpoints)**: Each endpoint shows which [FR-X.Y](#) requirement(s) it satisfies, which [US-XXX](#) story(ies) it implements, and which [TB-X](#) table(s) it accesses
- **TB-X (Database Tables)**: Each table shows which [FR-X.Y](#) requirement(s) it supports, which [EP-X](#) endpoint(s) use it, and which [US-XXX](#) story(ies) depend on it

**Quick Example**: Look at EP-1 (Register Guardian):
- ✅ **Requirement**: [FR-1.1](REQUIREMENTS.md#1-authentication--guardian-management)
- ✅ **User Story**: [US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp-parent-registration)
- ✅ **Database**: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

**For Complete Traceability**: See [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md) for a master map of all APIs, tables, requirements, and stories.

---

## Table of Contents

1. [SQLite Database Schema](#sqlite-database-schema)
   - **Phase 1 (MVP)**
     - [Authentication & Guardian Management](#core-tables)
       - [TB-1: Guardian Profile Table](#tb-1-guardian-profile-table)
       - [TB-2: Devices Table](#tb-2-devices-table)
       - [TB-3: Guardian Children Link Table](#tb-3-guardian-children-link-table)
     - [Dashboard & Student Profile](#tb-4-student-profile-table)
       - [TB-4: Student Profile Table](#tb-4-student-profile-table)
     - [Sync & Offline Functionality](#tb-5-sync-metadata-table)
       - [TB-5: Sync Metadata Table](#tb-5-sync-metadata-table)
     - [Attendance Intelligence](#tb-6-attendance-table)
       - [TB-6: Attendance Table](#tb-6-attendance-table)
     - [Homework & Assignment Tracker](#tb-7-homework-table)
       - [TB-7: Homework Table](#tb-7-homework-table)
     - [Examination & Result Management](#tb-8-exam-table)
       - [TB-8: Exam Table](#tb-8-exam-table)
       - [TB-9: Exam Results Table](#tb-9-exam-results-table)
     - [Daily Class Activity](#tb-10-class-activity-table)
       - [TB-10: Class Activity Table](#tb-10-class-activity-table)
     - [Class Routine](#tb-11-class-routine-table)
       - [TB-11: Class Routine Table](#tb-11-class-routine-table)
     - [School Information & Multi-Tenancy](#tb-12-school-info-table)
       - [TB-12: School Info Table](#tb-12-school-info-table)
       - [TB-13: School Contacts Table](#tb-13-school-contacts-table)
   - **Phase 2 & 3** — [Additional Tables (Phase 2 & 3)](#14-additional-tables-phase-2--3)
2. [REST API Endpoints](#rest-api-endpoints)
   - **Phase 1 (MVP)**
     - [Authentication & Guardian Management](#authentication-endpoints)
       - [EP-1: Register Guardian](#ep-1-register-guardian)
       - [EP-2: Guardian Login](#ep-2-guardian-login)
       - [EP-3: Send OTP (Child Linking)](#ep-3-send-otp-child-linking)
       - [EP-4: Verify OTP & Link Child](#ep-4-verify-otp--link-child)
       - [EP-5: Register Device](#ep-5-register-device)
     - [Sync & Offline Functionality](#sync-endpoints)
       - [EP-6: Initial Sync](#ep-6-initial-sync-first-login--new-device)
       - [EP-7: Delta Sync](#ep-7-delta-sync-10-minute-background)
     - [Dashboard & Student Profile](#dashboard--student-endpoints)
       - [EP-8: Get Dashboard](#ep-8-get-dashboard)
       - [EP-9: Get Guardian Children](#ep-9-get-guardian-children)
     - [Attendance Intelligence](#attendance-endpoints)
       - [EP-10: Get Attendance](#ep-10-get-attendance)
     - [Homework & Assignment Tracker](#homework-endpoints)
       - [EP-11: Get Homework List](#ep-11-get-homework-list)
     - [Examination & Result Management](#exam-endpoints)
       - [EP-12: Get Exam Schedule](#ep-12-get-exam-schedule)
       - [EP-13: Get Exam Results](#ep-13-get-exam-results)
     - [School Information & Multi-Tenancy](#school-endpoints)
       - [EP-14: Search Schools](#ep-14-search-schools)
       - [EP-15: Get School Directory](#ep-15-get-school-directory)
     - [Notifications](#notification-endpoints)
       - [EP-16: Get Notifications (Key Guardian only)](#ep-16-get-notifications)

---

## System Architecture Overview

### **High-Level Architecture**

```
┌──────────────────────────────────┐
│   Parent Mobile App (Flutter)    │
│  ┌──────────────────────────────┐│
│  │  UI Layer (Screens)          ││
│  │  - Dashboard, Attendance,     ││
│  │  - Homework, Exams, etc.      ││
│  └──────────────────────────────┘│
│  ┌──────────────────────────────┐│
│  │  Business Logic (Providers)  ││
│  │  - State management           ││
│  │  - Data transformations       ││
│  └──────────────────────────────┘│
│  ┌──────────────────────────────┐│
│  │  Data Layer                  ││
│  │  - API Service               ││
│  │  - SQLite Service            ││
│  └──────────────────────────────┘│
│  ┌──────────────────────────────┐│
│  │  Local Storage (SQLite)      ││
│  │  - All synced data           ││
│  │  - Offline-first             ││
│  └──────────────────────────────┘│
└──────────────────────────────────┘
         ↓ (HTTPS REST API)
┌──────────────────────────────────┐
│   Backend API (.NET Core)        │
│   (Out of Scope for this doc)    │
└──────────────────────────────────┘
         ↓
┌──────────────────────────────────┐
│   Backend Database               │
│   (Not in scope)                 │
└──────────────────────────────────┘
```

### **Key Principles**

1. **Offline-First**: App works without internet using local SQLite
2. **Delta Sync**: Only changed data synced (every 10 minutes)
3. **Multi-Tenancy**: Per-school isolation enforced at app level
4. **Device-Bound**: One account = one active device
5. **Responsive**: Sub-second local data access
6. **Reliable**: Graceful offline fallback, automatic retry

---

## Flutter Technology Stack

### **Core Frameworks & Libraries**

**State Management**:
- **Provider** (or Riverpod): Dependency injection & state management
  - Simplest learning curve
  - Good for small-medium apps
  - Clear separation of concerns
  
Alternative: **Riverpod** for immutability & testability

**Navigation**:
- **go_router**: Declarative routing
  - Type-safe navigation
  - Deep linking support
  - Nested navigation

**HTTP Client**:
- **Dio**: HTTP client with interceptors
  - Timeout handling
  - Request/response logging
  - Retry logic
  - Authentication interceptors

**Local Storage**:
- **sqflite**: SQLite wrapper for Flutter
  - Easy CRUD operations
  - Transaction support
  - Migration support
  
- **path_provider**: Access device file system (for database files)

**Notifications**:
- **firebase_messaging**: FCM for push notifications
  - Background message handling
  - Notification routing
  - Device token management

**JSON Serialization**:
- **json_serializable**: Code generation for JSON
  - Type-safe serialization
  - Null safety support
  
- **freezed**: Immutable model classes
  - Copy-with functionality
  - Type safety

**Localization**:
- **intl**: Internationalization support
  - Bangla (primary) & English (secondary)
  - Date/time formatting
  - Number formatting

**UI Components**:
- **Material Design**: Flutter Material package (built-in)
- **GetX** (optional): Toast, dialogs, snackbars (lightweight)

**Logging & Analytics**:
- **logger**: Structured logging for development
- **firebase_analytics**: Production analytics (optional)

**Testing**:
- **mockito**: Mock objects for testing
- **flutter_test**: Built-in testing framework
- **integration_test**: End-to-end testing

### **Development Tools**

- **Flutter SDK**: Latest stable (currently 3.16+)
- **Dart**: 3.0+
- **Android Studio** / **VS Code**: IDE
- **Android SDK**: API 26+ (Android 8.0)
- **Xcode**: iOS development (Xcode 14+)

### **Dependencies Summary**

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.0  # State management
  go_router: ^8.0.0  # Navigation
  dio: ^5.0.0  # HTTP client
  sqflite: ^2.2.0  # SQLite
  path_provider: ^2.0.0  # File paths
  firebase_messaging: ^14.0.0  # FCM
  json_serializable: ^6.0.0  # JSON
  freezed_annotation: ^2.0.0  # Immutable models
  intl: ^0.18.0  # Localization
  logger: ^1.4.0  # Logging
  shared_preferences: ^2.0.0  # Simple key-value store
  connectivity_plus: ^5.0.0  # Network state detection
  device_info_plus: ^9.0.0  # Device information
  package_info_plus: ^6.0.0  # App version info

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.0.0
  freezed: ^2.0.0
  json_serializable: ^6.0.0
  mockito: ^5.0.0
```

---

## Project Structure

### **Recommended Flutter Project Layout**

```
lib/
├── main.dart                      # App entry point
├── config/
│   ├── app_config.dart           # Environment config
│   ├── routes.dart               # Route definitions
│   └── theme.dart                # App theme & colors
├── core/
│   ├── constants/
│   │   ├── api_constants.dart    # API endpoints
│   │   ├── app_constants.dart    # App-wide constants
│   │   └── strings.dart          # Localized strings
│   ├── utils/
│   │   ├── date_utils.dart
│   │   ├── validators.dart
│   │   └── helpers.dart
│   └── errors/
│       ├── exceptions.dart       # Custom exceptions
│       └── failures.dart         # Failure handling
├── data/
│   ├── models/
│   │   ├── guardian_model.dart
│   │   ├── student_model.dart
│   │   ├── attendance_model.dart
│   │   └── ...                   # All other models
│   ├── datasources/
│   │   ├── local_datasource.dart # SQLite operations
│   │   └── remote_datasource.dart # API operations
│   ├── repositories/
│   │   ├── auth_repository.dart
│   │   ├── sync_repository.dart
│   │   ├── attendance_repository.dart
│   │   └── ...                   # All feature repos
│   └── database/
│       ├── database_provider.dart # SQLite setup
│       ├── migrations.dart        # Database migrations
│       └── schemas/
│           ├── guardian_schema.dart
│           ├── student_schema.dart
│           └── ...                # All table schemas
├── domain/
│   ├── usecases/                 # Business logic (if using Clean Architecture)
│   └── entities/                 # Pure data models
├── presentation/
│   ├── providers/                # State management (Provider/Riverpod)
│   │   ├── auth_provider.dart
│   │   ├── sync_provider.dart
│   │   ├── attendance_provider.dart
│   │   └── ...                   # All feature providers
│   ├── screens/
│   │   ├── auth/
│   │   │   ├── login_screen.dart
│   │   │   ├── register_screen.dart
│   │   │   └── add_child_screen.dart
│   │   ├── dashboard/
│   │   │   └── dashboard_screen.dart
│   │   ├── attendance/
│   │   │   └── attendance_screen.dart
│   │   └── ...                   # All screens by feature
│   ├── widgets/
│   │   ├── common/
│   │   │   ├── app_bar_widget.dart
│   │   │   ├── loading_widget.dart
│   │   │   └── error_widget.dart
│   │   └── feature_widgets/
│   │       ├── attendance_calendar.dart
│   │       ├── homework_list.dart
│   │       └── ...                # Reusable widgets
│   └── dialogs/
│       ├── otp_dialog.dart
│       ├── error_dialog.dart
│       └── ...                    # All dialogs
└── services/
    ├── api_service.dart          # HTTP client configuration
    ├── fcm_service.dart          # Firebase messaging
    ├── sync_service.dart         # Background sync
    ├── localization_service.dart # Language switching
    └── device_service.dart       # Device info
```

---

## SQLite Database Schema

### **Database Initialization**

```dart
// database_provider.dart
class DatabaseProvider {
  static final DatabaseProvider _instance = DatabaseProvider._internal();
  static Database? _database;

  factory DatabaseProvider() {
    return _instance;
  }

  DatabaseProvider._internal();

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initializeDatabase();
    return _database!;
  }

  Future<Database> _initializeDatabase() async {
    final databasePath = await getDatabasesPath();
    final path = join(databasePath, 'guardian_app.db');

    return await openDatabase(
      path,
      version: 1,
      onCreate: _createDatabase,
      onUpgrade: _upgradeDatabase,
    );
  }

  Future<void> _createDatabase(Database db, int version) async {
    // Create all tables in version order
    await db.execute(_guardianProfileSchema);
    await db.execute(_devicesSchema);
    await db.execute(_studentProfileSchema);
    await db.execute(_attendanceSchema);
    await db.execute(_homeworkSchema);
    await db.execute(_examSchema);
    await db.execute(_classActivitySchema);
    await db.execute(_classRoutineSchema);
    // ... all other schemas
    
    // Create indexes
    await _createIndexes(db);
  }

  Future<void> _upgradeDatabase(Database db, int oldVersion, int newVersion) async {
    // Handle migrations here
    // Version 1 -> 2: Add new columns, tables, etc.
  }
}
```

### **Core Tables**

#### **TB-1: Guardian Profile Table**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **API Endpoints**: [EP-1](#ep-1-register-guardian), [EP-2](#ep-2-guardian-login), [EP-5](#ep-5-register-device)

```sql
CREATE TABLE guardian_profile (
  guardian_id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  full_name TEXT NOT NULL,
  phone TEXT,
  role TEXT CHECK(role IN ('key', 'additional')),
  status TEXT CHECK(status IN ('active', 'revoked')) DEFAULT 'active',
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  last_login TEXT
);

CREATE INDEX idx_guardian_email ON guardian_profile(email);
```

#### **TB-2: Devices Table**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **API Endpoints**: [EP-5](#ep-5-register-device), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE devices (
  device_id TEXT PRIMARY KEY,
  guardian_id TEXT NOT NULL REFERENCES guardian_profile(guardian_id),
  fcm_token TEXT,
  brand TEXT,
  model TEXT,
  os_version TEXT,
  status TEXT CHECK(status IN ('active', 'locked')) DEFAULT 'active',
  registered_at TEXT NOT NULL,
  last_sync TEXT
);

CREATE INDEX idx_device_guardian ON devices(guardian_id);
```

#### **TB-3: Guardian Children Link Table**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **API Endpoints**: [EP-4](#ep-4-verify-otp--link-child), [EP-9](#ep-9-get-guardian-children)

```sql
CREATE TABLE guardian_children (
  guardian_id TEXT NOT NULL REFERENCES guardian_profile(guardian_id),
  child_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  relationship TEXT,
  linked_at TEXT NOT NULL,
  status TEXT CHECK(status IN ('active', 'revoked')) DEFAULT 'active',
  PRIMARY KEY (guardian_id, child_id, school_id)
);

CREATE INDEX idx_guardian_child ON guardian_children(guardian_id, child_id);
CREATE INDEX idx_child_school ON guardian_children(child_id, school_id);
```

#### **TB-4: Student Profile Table**
**Feature**: [Feature 3: Dashboard](#3-dashboard--student-profile) | **API Endpoints**: [EP-8](#ep-8-get-dashboard), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE student_profile (
  student_id TEXT PRIMARY KEY,
  student_id_number TEXT,
  name TEXT NOT NULL,
  dob TEXT NOT NULL,
  photo_url TEXT,
  gender TEXT,
  class TEXT NOT NULL,
  section TEXT,
  curriculum TEXT,
  current_session TEXT,
  address TEXT,
  school_id TEXT NOT NULL,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX idx_student_school ON student_profile(school_id);
```

#### **TB-5: Sync Metadata Table**
**Feature**: [Feature 2: Sync & Offline](#2-sync--offline-functionality) | **API Endpoints**: [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE sync_metadata (
  school_id TEXT PRIMARY KEY,
  last_sync_datetime TEXT,
  last_sync_status TEXT,
  last_sync_error TEXT,
  last_sync_time TEXT,
  device_locked INTEGER DEFAULT 0,
  created_at TEXT NOT NULL
);
```

#### **TB-6: Attendance Table**
**Feature**: [Feature 5: Attendance](#5-attendance-intelligence) | **API Endpoints**: [EP-10](#ep-10-get-attendance), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE attendance (
  attendance_id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  attendance_date TEXT NOT NULL,
  status TEXT CHECK(status IN ('present', 'absent', 'leave', 'half_day')) NOT NULL,
  duration_minutes INTEGER,
  check_in_time TEXT,
  check_out_time TEXT,
  remarks TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  UNIQUE(student_id, school_id, attendance_date)
);

CREATE INDEX idx_attendance_student_date ON attendance(student_id, attendance_date);
CREATE INDEX idx_attendance_school ON attendance(school_id);
```

#### **TB-7: Homework Table**
**Feature**: [Feature 8: Homework](#8-homework--assignment-tracker) | **API Endpoints**: [EP-11](#ep-11-get-homework-list), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE homework (
  homework_id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  subject TEXT NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  assigned_date TEXT NOT NULL,
  due_date TEXT NOT NULL,
  status TEXT CHECK(status IN ('assigned', 'submitted', 'evaluated', 'pending', 'late', 'overdue', 'void')),
  marks INTEGER,
  total_marks INTEGER,
  grade TEXT,
  feedback TEXT,
  teacher_id TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX idx_homework_student_due ON homework(student_id, due_date);
CREATE INDEX idx_homework_subject ON homework(subject);
```

#### **TB-8: Exam Table**
**Feature**: [Feature 9: Examination](#9-examination--result-management) | **API Endpoints**: [EP-12](#ep-12-get-exam-schedule), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE exam (
  exam_id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  exam_type TEXT NOT NULL,
  subject TEXT NOT NULL,
  exam_date TEXT NOT NULL,
  exam_time TEXT,
  duration_minutes INTEGER,
  instructions TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX idx_exam_student_date ON exam(student_id, exam_date);
CREATE INDEX idx_exam_subject ON exam(subject);
```

#### **TB-9: Exam Results Table**
**Feature**: [Feature 9: Examination](#9-examination--result-management) | **API Endpoints**: [EP-13](#ep-13-get-exam-results), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE exam_result (
  result_id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  exam_id TEXT REFERENCES exam(exam_id),
  subject TEXT NOT NULL,
  marks_obtained REAL NOT NULL,
  total_marks REAL NOT NULL,
  grade TEXT,
  percentile REAL,
  rank INTEGER,
  class_average REAL,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX idx_result_student_exam ON exam_result(student_id, exam_id);
```

#### **TB-10: Class Activity Table**
**Feature**: [Feature 7: Class Activity](#7-daily-class-activity) | **API Endpoints**: [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE class_activity (
  activity_id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  activity_date TEXT NOT NULL,
  subject TEXT NOT NULL,
  topics_taught TEXT,
  chapters_covered TEXT,
  activities TEXT,
  remarks TEXT,
  teacher_id TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX idx_activity_student_date ON class_activity(student_id, activity_date);
CREATE INDEX idx_activity_subject ON class_activity(subject);
```

#### **TB-11: Class Routine Table**
**Feature**: [Feature 6: Class Routine](#6-class-routine) | **API Endpoints**: [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE class_routine (
  routine_id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  school_id TEXT NOT NULL,
  day_of_week INTEGER NOT NULL,
  period INTEGER NOT NULL,
  subject TEXT NOT NULL,
  teacher_id TEXT,
  teacher_name TEXT,
  start_time TEXT,
  end_time TEXT,
  location TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  UNIQUE(student_id, school_id, day_of_week, period)
);

CREATE INDEX idx_routine_student ON class_routine(student_id, school_id);
```

#### **TB-12: School Info Table**
**Feature**: [Feature 4: School](#4-school-information--multi-tenancy) | **API Endpoints**: [EP-14](#ep-14-search-schools), [EP-15](#ep-15-get-school-directory), [EP-6](#ep-6-initial-sync), [EP-7](#ep-7-delta-sync)

```sql
CREATE TABLE school_info (
  school_id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  code TEXT,
  logo_url TEXT,
  primary_color TEXT,
  secondary_color TEXT,
  accent_color TEXT,
  address TEXT,
  phone TEXT,
  email TEXT,
  website_url TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);
```

#### **TB-13: School Contacts Table**
**Feature**: [Feature 4: School](#4-school-information--multi-tenancy) | **API Endpoints**: [EP-15](#ep-15-get-school-directory)

```sql
CREATE TABLE school_contacts (
  contact_id TEXT PRIMARY KEY,
  school_id TEXT NOT NULL REFERENCES school_info(school_id),
  role TEXT CHECK(role IN ('principal', 'admin', 'teacher', 'counselor', 'support')),
  name TEXT NOT NULL,
  title TEXT,
  subject TEXT,
  email TEXT,
  phone TEXT,
  office_hours TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX idx_contact_school_role ON school_contacts(school_id, role);
```

#### **14. Additional Tables** (Phase 2 & 3)

Health data, vaccination, behavior score, notice board, events, fees, notifications, etc.

[All tables follow same pattern with timestamps, indexes, and proper relationships]

### **Data Models (Dart)**

```dart
// models/guardian_model.dart
@freezed
class Guardian with _$Guardian {
  const factory Guardian({
    required String guardianId,
    required String email,
    required String fullName,
    String? phone,
    required GuardianRole role,
    required GuardianStatus status,
    required DateTime createdAt,
    required DateTime updatedAt,
  }) = _Guardian;

  factory Guardian.fromJson(Map<String, dynamic> json) =>
      _$GuardianFromJson(json);
}

enum GuardianRole { key, additional }
enum GuardianStatus { active, revoked }
```

---

## REST API Endpoints

### **Base Configuration**

```
Base URL: https://api.parentapp.edu.bd/api/v1
Timeout: 120 seconds (maximum per requirement)
Retry Logic: Exponential backoff, max 3 attempts
Authentication: Bearer {JWT_TOKEN}
```

### **Authentication Endpoints**

#### **EP-1: Register Guardian**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **User Stories**: [US-001-MVP](#us-001-mvp), [US-002-MVP](#us-002-mvp)
```http
POST /api/v1/auth/register
Content-Type: application/json

Request Body:
{
  "full_name": "Ahmed Hassan",
  "email": "ahmed@example.com",
  "password": "SecurePass123",
  "confirm_password": "SecurePass123"
}

Response 201 Created:
{
  "success": true,
  "code": "REGISTRATION_SUCCESS",
  "message": "Account created successfully",
  "data": {
    "guardian_id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "ahmed@example.com",
    "created_at": "2026-05-23T10:30:00Z"
  },
  "timestamp": "2026-05-23T10:30:00Z"
}

Response 400 Bad Request:
{
  "success": false,
  "code": "VALIDATION_ERROR",
  "message": "Email format invalid",
  "timestamp": "2026-05-23T10:30:00Z"
}

Response 409 Conflict:
{
  "success": false,
  "code": "EMAIL_EXISTS",
  "message": "An account with this email already exists",
  "timestamp": "2026-05-23T10:30:00Z"
}
```

#### **EP-2: Guardian Login**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **User Stories**: [US-002-MVP](#us-002-mvp), [US-006-MVP](#us-006-mvp)
```http
POST /api/v1/auth/login
Content-Type: application/json

Request Body:
{
  "email": "ahmed@example.com",
  "password": "SecurePass123"
}

Response 200 OK:
{
  "success": true,
  "code": "LOGIN_SUCCESS",
  "message": "Login successful",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "guardian_id": "550e8400-e29b-41d4-a716-446655440000",
    "children": [
      {
        "child_id": "660e8400-e29b-41d4-a716-446655440000",
        "name": "Fatima Hassan",
        "school_id": "770e8400-e29b-41d4-a716-446655440000",
        "school_name": "Spring Valley School"
      }
    ],
    "token_expires_in": 86400,
    "remember_me_available": true
  }
}

Response 401 Unauthorized:
{
  "success": false,
  "code": "INVALID_CREDENTIALS",
  "message": "Incorrect email or password"
}

Response 401 Account Locked:
{
  "success": false,
  "code": "ACCOUNT_LOCKED",
  "message": "Too many failed attempts. Try again in 15 minutes.",
  "data": {
    "retry_after": 900,
    "remaining_wait_seconds": 847
  }
}

Response 403 Access Revoked:
{
  "success": false,
  "code": "ACCESS_REVOKED",
  "message": "Your access has been revoked. Contact the key guardian."
}
```

#### **EP-3: Send OTP (Child Linking)**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **User Stories**: [US-003-MVP](#us-003-mvp), [US-004-MVP](#us-004-mvp)
```http
POST /api/v1/children/otp/send
Authorization: Bearer {access_token}
Content-Type: application/json

Request Body:
{
  "student_id": "STU001234",
  "date_of_birth": "2015-06-15"
}

Response 200 OK:
{
  "success": true,
  "code": "OTP_SENT",
  "message": "OTP sent to your registered phone",
  "data": {
    "otp_sent_to": "+880 1234 ****56",
    "resend_available_after": 30,
    "expires_at": "2026-05-23T11:10:00Z",
    "otp_length": 6
  }
}

Response 400 Bad Request:
{
  "success": false,
  "code": "INVALID_STUDENT",
  "message": "Student ID and Date of Birth do not match"
}

Response 403 Forbidden:
{
  "success": false,
  "code": "NOT_KEY_GUARDIAN",
  "message": "Your email does not match the key guardian email"
}
```

#### **EP-4: Verify OTP & Link Child**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **User Stories**: [US-003-MVP](#us-003-mvp), [US-005-MVP](#us-005-mvp)
```http
POST /api/v1/children/otp/verify
Authorization: Bearer {access_token}
Content-Type: application/json

Request Body:
{
  "student_id": "STU001234",
  "otp_code": "123456"
}

Response 200 OK:
{
  "success": true,
  "code": "OTP_VERIFIED",
  "message": "Child linked successfully",
  "data": {
    "child_id": "660e8400-e29b-41d4-a716-446655440000",
    "school_id": "770e8400-e29b-41d4-a716-446655440000",
    "next_step": "device_register",
    "linked_at": "2026-05-23T11:05:00Z"
  }
}

Response 400 Bad Request:
{
  "success": false,
  "code": "INVALID_OTP",
  "message": "Incorrect code. Try again.",
  "data": {
    "attempt": 1,
    "remaining_attempts": 2
  }
}

Response 400 Max Attempts:
{
  "success": false,
  "code": "MAX_OTP_ATTEMPTS",
  "message": "Too many attempts. Resend the code.",
  "data": {
    "retry_after": 30
  }
}
```

#### **EP-5: Register Device**
**Feature**: [Feature 1: Authentication](#1-authentication--guardian-management) | **User Stories**: [US-006-MVP](#us-006-mvp)
```http
POST /api/v1/auth/device-register
Authorization: Bearer {access_token}
Content-Type: application/json

Request Body:
{
  "guardian_id": "550e8400-e29b-41d4-a716-446655440000",
  "device_id": "device-uuid-123",
  "fcm_token": "firebase-cloud-messaging-token",
  "brand": "Samsung",
  "model": "Galaxy S21",
  "os_version": "Android 13"
}

Response 200 OK:
{
  "success": true,
  "code": "DEVICE_REGISTERED",
  "message": "Device registered successfully",
  "data": {
    "device_id": "device-uuid-123",
    "registered_at": "2026-05-23T11:05:00Z",
    "status": "active"
  }
}

Response 409 Conflict:
{
  "success": false,
  "code": "DEVICE_ALREADY_REGISTERED",
  "message": "Device already registered to another guardian"
}
```

### **Sync Endpoints**

#### **EP-6: Initial Sync (First Login / New Device)**
**Feature**: [Feature 2: Sync & Offline](#2-sync--offline-functionality) | **User Stories**: [US-009-MVP](#us-009-mvp), [US-010-MVP](#us-010-mvp)
```http
POST /api/v1/sync/initial-login
Authorization: Bearer {access_token}
X-Device-ID: device-uuid-123
Content-Type: application/json

Request Body:
{
  "school_id": "770e8400-e29b-41d4-a716-446655440000",
  "guardian_id": "550e8400-e29b-41d4-a716-446655440000",
  "child_ids": ["660e8400-e29b-41d4-a716-446655440000"],
  "sync_type": "initial",
  "device_id": "device-uuid-123",
  "current_datetime": "2026-05-23T10:30:00Z"
}

Response 200 OK:
{
  "success": true,
  "code": "INITIAL_SYNC_COMPLETE",
  "data": {
    "sync_timestamp": "2026-05-23T10:30:00Z",
    "sync_type": "initial",
    "school_info": {
      "school_id": "770e8400-e29b-41d4-a716-446655440000",
      "name": "Spring Valley School",
      "logo_url": "https://...",
      "theme": {
        "primary_color": "#1976D2",
        "secondary_color": "#DC004E"
      }
    },
    "children": [
      {
        "child_id": "660e8400-e29b-41d4-a716-446655440000",
        "student_profile": {
          "student_id": "STU001234",
          "name": "Fatima Hassan",
          "photo_url": "https://...",
          "dob": "2015-06-15",
          "class": "IX",
          "section": "A"
        },
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
          "attendance": [...]
        }
      }
    ]
  }
}

Response 401 Unauthorized:
{
  "success": false,
  "code": "UNAUTHORIZED",
  "message": "Invalid or expired token"
}

Response 403 Forbidden:
{
  "success": false,
  "code": "ACCESS_DENIED",
  "message": "Child not linked to your account"
}
```

#### **EP-7: Delta Sync (10-Minute Background)**
**Feature**: [Feature 2: Sync & Offline](#2-sync--offline-functionality) | **User Stories**: [US-010-MVP](#us-010-mvp), [US-011-MVP](#us-011-mvp)
```http
POST /api/v1/sync/delta
Authorization: Bearer {access_token}
X-Device-ID: device-uuid-123
Content-Type: application/json

Request Body:
{
  "school_id": "770e8400-e29b-41d4-a716-446655440000",
  "guardian_id": "550e8400-e29b-41d4-a716-446655440000",
  "child_ids": ["660e8400-e29b-41d4-a716-446655440000"],
  "sync_type": "delta",
  "device_id": "device-uuid-123",
  "last_sync_datetime": "2026-05-23T10:20:00Z",
  "current_datetime": "2026-05-23T10:30:00Z"
}

Response 200 OK:
{
  "success": true,
  "code": "DELTA_SYNC_COMPLETE",
  "data": {
    "sync_timestamp": "2026-05-23T10:30:00Z",
    "sync_type": "delta",
    "school_id": "770e8400-e29b-41d4-a716-446655440000",
    "revocation_status": {
      "is_revoked": false,
      "reason": null
    },
    "children": [
      {
        "child_id": "660e8400-e29b-41d4-a716-446655440000",
        "changes": {
          "homework": [
            {
              "homework_id": "hw-uuid-001",
              "subject": "Mathematics",
              "title": "Chapter 5: Equations",
              "due_date": "2026-05-24",
              "status": "assigned",
              "updated_at": "2026-05-23T10:28:00Z"
            }
          ]
        },
        "void_records": {
          "homework": [
            {
              "homework_id": "hw-uuid-999",
              "status": "void"
            }
          ]
        }
      }
    ]
  }
}

Response 403 Device Locked:
{
  "success": false,
  "code": "DEVICE_LOCKED",
  "message": "You logged in from another device. Session ended.",
  "data": {
    "action": "logout"
  }
}

Response 403 Revoked:
{
  "success": true,
  "code": "REVOKED_GUARDIAN",
  "data": {
    "sync_timestamp": "2026-05-23T10:31:00Z",
    "revocation_status": {
      "is_revoked": true,
      "reason": "Your access has been revoked by the key guardian"
    }
  }
}
```

### **Dashboard & Student Endpoints**

#### **EP-8: Get Dashboard**
**Feature**: [Feature 3: Dashboard](#3-dashboard--student-profile) | **User Stories**: [US-015-MVP](#us-015-mvp), [US-016-MVP](#us-016-mvp)
```http
GET /api/v1/dashboard/{child_id}
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "child_id": "660e8400-e29b-41d4-a716-446655440000",
    "student": {
      "name": "Fatima Hassan",
      "photo_url": "https://...",
      "class": "IX",
      "section": "A"
    },
    "school": {
      "name": "Spring Valley School",
      "logo_url": "https://..."
    },
    "quick_stats": {
      "attendance_percent": 92.5,
      "next_homework": {
        "subject": "Mathematics",
        "title": "Chapter 5 Exercises",
        "due_date": "2026-05-24",
        "days_remaining": 1
      },
      "next_exam": {
        "date": "2026-06-15",
        "subject": "English"
      },
      "alerts": {
        "count": 0,
        "messages": []
      }
    }
  }
}
```

#### **EP-9: Get Guardian Children**
**Feature**: [Feature 3: Dashboard](#3-dashboard--student-profile) | **User Stories**: [US-018-MVP](#us-018-mvp)
```http
GET /api/v1/guardian/children
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "children": [
      {
        "child_id": "660e8400-e29b-41d4-a716-446655440000",
        "name": "Fatima Hassan",
        "school_name": "Spring Valley School",
        "class": "IX",
        "status": "active"
      }
    ],
    "last_selected_child_id": "660e8400-e29b-41d4-a716-446655440000"
  }
}
```

### **Attendance Endpoints**

#### **EP-10: Get Attendance**
**Feature**: [Feature 5: Attendance](#5-attendance-intelligence) | **User Stories**: [US-022-MVP](#us-022-mvp), [US-023-MVP](#us-023-mvp), [US-024-MVP](#us-024-mvp)
```http
GET /api/v1/attendance/{child_id}?month=5&year=2026
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "child_id": "660e8400-e29b-41d4-a716-446655440000",
    "month": 5,
    "year": 2026,
    "summary": {
      "total_days": 22,
      "present_days": 20,
      "absent_days": 1,
      "leave_days": 1,
      "attendance_percent": 92.27
    },
    "daily_records": [
      {
        "date": "2026-05-01",
        "status": "present",
        "remarks": null
      }
    ],
    "trends": {
      "previous_month_percent": 90.5,
      "change_percent": 1.77
    },
    "alerts": []
  }
}
```

### **Homework Endpoints**

#### **EP-11: Get Homework List**
**Feature**: [Feature 8: Homework](#8-homework--assignment-tracker) | **User Stories**: [US-027-MVP](#us-027-mvp), [US-028-MVP](#us-028-mvp), [US-029-MVP](#us-029-mvp)
```http
GET /api/v1/homework/{child_id}?filter=all&sort=due_date
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "homework": [
      {
        "homework_id": "hw-uuid-001",
        "subject": "Mathematics",
        "title": "Chapter 5 Exercises",
        "due_date": "2026-05-24",
        "status": "assigned",
        "marks": null,
        "feedback": null,
        "updated_at": "2026-05-23T10:28:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45
    }
  }
}
```

### **Exam Endpoints**

#### **EP-12: Get Exam Schedule**
**Feature**: [Feature 9: Examination](#9-examination--result-management) | **User Stories**: [US-030-MVP](#us-030-mvp)
```http
GET /api/v1/exams/{child_id}?status=upcoming
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "exams": [
      {
        "exam_id": "exam-uuid-001",
        "exam_type": "Monthly Test",
        "subject": "Mathematics",
        "date": "2026-06-15",
        "time": "09:30",
        "duration_minutes": 60
      }
    ]
  }
}
```

#### **EP-13: Get Exam Results**
**Feature**: [Feature 9: Examination](#9-examination--result-management) | **User Stories**: [US-031-MVP](#us-031-mvp), [US-032-MVP](#us-032-mvp)
```http
GET /api/v1/exam-results/{child_id}?limit=5
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "results": [
      {
        "result_id": "result-uuid-001",
        "exam_id": "exam-uuid-001",
        "subject": "Mathematics",
        "marks_obtained": 85,
        "total_marks": 100,
        "grade": "A",
        "percentile": 85,
        "rank": 5,
        "class_average": 72.5
      }
    ]
  }
}
```

### **School Endpoints**

#### **EP-14: Search Schools**
**Feature**: [Feature 4: School](#4-school-information--multi-tenancy) | **User Stories**: [US-019-MVP](#us-019-mvp)
```http
GET /api/v1/schools/search?query=Spring
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "schools": [
      {
        "school_id": "770e8400-e29b-41d4-a716-446655440000",
        "name": "Spring Valley School",
        "code": "SVS001",
        "city": "Dhaka",
        "logo_url": "https://...",
        "contact_email": "info@springvalley.edu.bd"
      }
    ],
    "total": 1
  }
}
```

#### **EP-15: Get School Directory**
**Feature**: [Feature 4: School](#4-school-information--multi-tenancy) | **User Stories**: [US-020-MVP](#us-020-mvp)
```http
GET /api/v1/schools/{school_id}/directory
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "principal": {
      "name": "Dr. Karim Khan",
      "phone": "+880-1234-567890",
      "email": "principal@school.edu.bd"
    },
    "administration": [...],
    "class_teachers": [...],
    "support_staff": [...]
  }
}
```

### **Notification Endpoints**

#### **EP-16: Get Notifications**
**Feature**: [Feature 10: Notifications](REQUIREMENTS.md#10-notifications-core-implementation) | **User Stories**: [US-030-MVP](USER_STORIES_PERSONAS.md#us-030-mvp-push-notifications-for-key-events), [US-031-MVP](USER_STORIES_PERSONAS.md#us-031-mvp-notification-preferences), [US-032-MVP](USER_STORIES_PERSONAS.md#us-032-mvp-notification-history)  
**Access**: Key Guardian only — FCM push and notification history are not available to Additional Guardian
```http
GET /api/v1/notifications?page=1&limit=20
Authorization: Bearer {access_token}

Response 200 OK:
{
  "success": true,
  "data": {
    "notifications": [
      {
        "notification_id": "notif-uuid-001",
        "type": "homework_due",
        "title": "Mathematics homework due tomorrow",
        "message": "Chapter 5 Exercises",
        "read": false,
        "created_at": "2026-05-23T10:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150
    }
  }
}
```

---

## Data Sync Flow

### **10-Minute Background Sync Sequence**

```
┌─────────────────────────────────────────────────────────┐
│ Timer: Every 10 minutes when app is running             │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ Check Network & Device Status                           │
│ - Is device online? (connectivity_plus)                 │
│ - Is device_id still active? (not locked)               │
└──────────────────┬──────────────────────────────────────┘
                   ↓
          Yes        ↓        No
         ────────────┴─────────────
         ↓                    ↓
      Continue           Skip sync,
      to sync           wait for next
                        10-minute cycle
                   ↓
┌─────────────────────────────────────────────────────────┐
│ Prepare Sync Request (Per School)                       │
│ - Get last_sync_datetime from sync_metadata             │
│ - Build request with school_id, child_ids, timestamps   │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ POST /api/v1/sync/delta                                 │
│ - Send request with timeout (120 seconds max)           │
│ - Attach authorization token                            │
└──────────────────┬──────────────────────────────────────┘
                   ↓
         Response received
                   ↓
    ┌──────────┴──────────┬──────────┬──────────┐
    ↓                     ↓          ↓          ↓
  200 OK             401/403       5xx       Timeout
   ↓                   ↓           ↓          ↓
Merge data        Handle auth    Retry at   Retry at
                  failure        next cycle next cycle
                   ↓                ↓          ↓
                 Logout          Log error  Log error
                   ↓                ↓          ↓
┌─────────────────────────────────────────────────────────┐
│ Merge Response into Local SQLite                        │
│ - FOR each child in response:                           │
│   - FOR each data_type in changes:                      │
│     - IF record exists: UPDATE                          │
│     - IF new record: INSERT                             │
│   - FOR each void_record:                               │
│     - UPDATE status = 'void'                            │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ Update sync_metadata                                    │
│ - last_sync_datetime = response.sync_timestamp          │
│ - last_sync_status = 'success'                          │
│ - last_sync_error = NULL                                │
│ - last_sync_time = NOW()                                │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ Check Revocation                                        │
│ - IF revocation_status.is_revoked == true:              │
│   - Clear all local data                                │
│   - Clear JWT token                                     │
│   - Force logout                                        │
│   - Show message                                        │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ Notify UI: Data Updated                                 │
│ - Trigger UI rebuild                                    │
│ - Show "Last synced: just now"                          │
│ - Update all screens with new data                      │
└─────────────────────────────────────────────────────────┘
```

---

## Error Handling & Retry Logic

### **HTTP Error Handling**

```dart
// api_service.dart
class ApiService {
  Future<T> request<T>(
    String method,
    String endpoint, {
    Map<String, dynamic>? body,
    Map<String, dynamic>? queryParams,
    int maxRetries = 3,
  }) async {
    int attempt = 0;
    Duration delay = Duration(seconds: 1);

    while (attempt < maxRetries) {
      try {
        final response = await _dio.request(
          endpoint,
          options: Options(method: method),
          data: body,
          queryParameters: queryParams,
        );

        return _handleSuccess<T>(response);
      } on DioException catch (e) {
        attempt++;
        
        if (attempt >= maxRetries) {
          return _handleError<T>(e);
        }

        // Exponential backoff
        await Future.delayed(delay);
        delay *= 2;
      }
    }
  }

  T _handleSuccess<T>(Response response) {
    if (response.statusCode == 200 || response.statusCode == 201) {
      // Parse JSON response
      final data = response.data['data'];
      return T.fromJson(data);
    }
    throw ApiException('Unexpected status code: ${response.statusCode}');
  }

  _handleError<T>(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
        throw NetworkException('Connection timeout');
      case DioExceptionType.receiveTimeout:
        throw NetworkException('Receive timeout');
      case DioExceptionType.sendTimeout:
        throw NetworkException('Send timeout');
      case DioExceptionType.badResponse:
        // Handle specific HTTP errors
        final statusCode = error.response?.statusCode;
        if (statusCode == 401) {
          throw UnauthorizedException('Invalid token');
        } else if (statusCode == 403) {
          throw ForbiddenException('Access denied');
        } else if (statusCode >= 500) {
          throw ServerException('Server error');
        }
        break;
      default:
        throw ApiException('Unknown error: ${error.message}');
    }
  }
}
```

### **Sync Error Scenarios**

| Error | Handling |
|-------|----------|
| **Network timeout** | Retry at next 10-min cycle; show offline indicator |
| **Server error (5xx)** | Log error, retry at next cycle; after 5 failures, show alert |
| **401 Unauthorized** | Force logout immediately; redirect to login |
| **403 Device locked** | Force logout; show "Session ended from another device" |
| **403 Access revoked** | Force logout; show "Your access has been revoked" |
| **Validation error (400)** | Log error, skip this sync; show sync failed notification |

---

## References

- [REQUIREMENTS.md](REQUIREMENTS.md) - Feature specifications and requirements
- [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) - User stories and acceptance criteria
- [INDEX.md](INDEX.md) - Document navigation

---

**Document Version**: 1.0  
**Last Updated**: 2026-05-23  
**Status**: ✅ Final - Ready for Flutter Development

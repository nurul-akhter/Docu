# Guardian Parent App - Complete Documentation Index

> **Project**: Guardian Parent App (Mobile-First SaaS Platform)  
> **Target Market**: Bangladesh Schools  
> **Platforms**: Flutter (Android, iOS)  
> **Status**: Requirements & Design Phase  
> **Last Updated**: 2026-05-23

---

## 📚 Documentation Overview

This project is documented across **5 integrated documents** with full bidirectional cross-references. Each serves a specific purpose and audience. Use the navigation below to find what you need.

```
┌──────────────────────────────────────────────────────────────┐
│  INDEX.md (You are here)                                     │
│  Master navigation & document relationships                  │
└──────┬────────────────┬──────────────┬──────────────┬────────┘
       │                │              │              │
   ┌───▼───┐      ┌────▼─────┐  ┌────▼──────┐   ┌───▼─────┐
   │ User  │      │Requirements    │ Tech      │   │Traceability
   │Stories│      │ Document   │ Architecture  │   │Matrix
   │Personas    │             │               │   │
   └───────┘      └──────────┘  └──────────┘   └─────────┘
```

---

## 🎯 Quick Navigation by Role

### **For Product Managers & Business Analysts**
Start here → [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md)
- Who are our users? (Personas)
- What do they want to do? (User Stories)
- How do we prioritize? (Backlog with priorities)
- When do we build it? (Phased approach)

Then → [REQUIREMENTS.md](REQUIREMENTS.md) → Features & Acceptance Criteria

---

### **For Developers & Architects**
Start here → [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)
- Flutter architecture & tech stack
- SQLite database schema (local storage)
- REST API endpoint contracts
- How to call the backend

Then → [REQUIREMENTS.md](REQUIREMENTS.md) → Non-functional requirements & constraints

---

### **For QA & Testing Teams**
Start here → [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md#acceptance-criteria)
- Acceptance criteria for each story
- Test scenarios

Then → [REQUIREMENTS.md](REQUIREMENTS.md) → Functional requirements for test cases

Then → [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md) → API contracts for integration testing

---

### **For Project Managers**
Start here → [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md#backlog-summary)
- Story points (effort estimation)
- Dependencies (critical path)
- Phases & timelines

Then → [REQUIREMENTS.md](REQUIREMENTS.md#system-wide-requirements) → Non-functional requirements & scaling

---

### **For Stakeholders & Executives**
Read → [REQUIREMENTS.md](REQUIREMENTS.md#executive-summary) → Executive Summary section only

---

## 📋 Document Descriptions

### **1. [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md)** 
**Purpose**: Define WHO we're building for and WHAT they want  
**Audience**: Product Managers, Scrum Masters, UX Designers  
**Contains**:
- 👤 **User Personas** (Guardian, Key Guardian, Additional Guardian)
- 📖 **User Stories Backlog** (~80-100 stories)
  - Story ID, Epic, Phase
  - User story format: "As a [role], I want [action], so that [benefit]"
  - Acceptance criteria
  - Story points (effort: XS, S, M, L, XL)
  - Priority (Must-have, Should-have, Nice-to-have)
  - Dependencies
  - Status
- 📊 **Backlog Summary** (organized by phase & priority)

**Key Sections**:
- [Personas](USER_STORIES_PERSONAS.md#personas)
- [Phase 1 (MVP) Backlog](USER_STORIES_PERSONAS.md#phase-1-mvp-backlog)
- [Phase 2 (Enhanced) Backlog](USER_STORIES_PERSONAS.md#phase-2-enhanced-backlog)
- [Phase 3 (Health) Backlog](USER_STORIES_PERSONAS.md#phase-3-health--growth-backlog)

---

### **2. [REQUIREMENTS.md](REQUIREMENTS.md)**
**Purpose**: Define WHAT we need to build  
**Audience**: All team members (comprehensive reference)  
**Contains**:
- 📌 **Feature Requirements** (28 features across 3 phases)
  - For each feature:
    - Overview & context
    - Functional Requirements (FR)
    - Non-Functional Requirements (NFR)
    - Acceptance Criteria
    - References to User Stories & API contracts
- 🎯 **System-Wide Requirements**
  - Performance targets (API ≤ 120 sec, 99% uptime)
  - Security & data privacy
  - Scaling: 5,000 → 10,000 users
  - Device support (Android 8+, iOS 13+)
- 📊 **Data Classification** (Academic-year vs. Continuous)

**Key Sections**:
- [Executive Summary](REQUIREMENTS.md#executive-summary)
- [Feature 1: Authentication](REQUIREMENTS.md#1-authentication--guardian-management)
- [Feature 2: Sync & Offline](REQUIREMENTS.md#2-sync--offline-functionality)
- [All 28 Features](REQUIREMENTS.md#table-of-contents)
- [System-Wide NFRs](REQUIREMENTS.md#system-wide-non-functional-requirements)

---

### **3. [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)**
**Purpose**: Define HOW we technically build it  
**Audience**: Developers, Architects, DevOps, Backend team  
**Contains**:
- 🏗️ **Architecture Overview**
  - High-level system diagram
  - Mobile app ↔ API ↔ Backend flow
  - Offline-first strategy
- 🐦 **Flutter Architecture**
  - Technology stack
  - State management choice
  - Key packages
  - Project structure
- 📦 **SQLite Database Schema**
  - All table definitions (CREATE TABLE)
  - Relationships, indexes, constraints
  - Tables organized by feature
  - Local storage structure
  - Sync metadata tracking
- 🔌 **REST API Endpoints**
  - Organized by feature/module
  - For each endpoint:
    - HTTP method & path
    - Authentication requirements
    - Request parameters & body
    - Response format & status codes
    - Example request/response
    - Error handling

**Key Sections**:
- [System Architecture](TECH_ARCHITECTURE.md#system-architecture)
- [Flutter Stack](TECH_ARCHITECTURE.md#flutter-technology-stack)
- [SQLite Schema](TECH_ARCHITECTURE.md#sqlite-database-schema)
- [API Endpoints](TECH_ARCHITECTURE.md#rest-api-endpoints)
  - [Authentication Endpoints](TECH_ARCHITECTURE.md#authentication-endpoints)
  - [Sync Endpoints](TECH_ARCHITECTURE.md#sync-endpoints)
  - [All Feature Endpoints](TECH_ARCHITECTURE.md#api-endpoints-by-feature)

---

### **5. [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md)**
**Purpose**: Map all bidirectional relationships between requirements, stories, APIs, and databases  
**Audience**: Developers, Architects, QA, Project Managers  
**Contains**:
- 🔗 **Cross-Reference ID Format** (FR-X.Y, US-XXX-PHASE, EP-X, TB-X)
- 📊 **Phase 1 (MVP) Feature Mapping**
  - Which user stories implement each requirement
  - Which API endpoints satisfy each requirement
  - Which database tables support each requirement
- 📋 **API Endpoint Master List** (EP-1 through EP-16)
- 🗄️ **Database Table Master List** (TB-1 through TB-13)
- ✅ **How to Use** (Trace any artifact to its related implementations)

**Key Sections**:
- [Reference ID Conventions](TRACEABILITY_MATRIX.md#reference-id-conventions)
- [Phase 1 (MVP) Feature Traceability](TRACEABILITY_MATRIX.md#phase-1-mvp---feature-traceability)
- [API Endpoint Master List](TRACEABILITY_MATRIX.md#api-endpoint-master-list)
- [Database Table Master List](TRACEABILITY_MATRIX.md#database-table-master-list)
- [How to Use This Matrix](TRACEABILITY_MATRIX.md#how-to-use-this-matrix)

---

## 🔗 How Documents Relate

```
USER PERSONA (WHO)
    ↓
    └─→ "I need to register and add my child"
        ↓
USER STORY (WHAT & WHY)
    ↓
    └─→ "As a parent, I want to register with email/password so I can create account"
        ↓
REQUIREMENT (FUNCTIONAL & NON-FUNCTIONAL)
    ↓
    └─→ FR: "Email validation, password hashing, confirmation email sent"
        └─→ NFR: "Registration response ≤ 2 seconds"
        ↓
API CONTRACT (HOW TO CALL BACKEND)
    ↓
    └─→ POST /api/v1/auth/register
        └─→ Body: { full_name, email, password, confirm_password }
        └─→ Response: 201 Created with guardian_id
        ↓
DATABASE SCHEMA (HOW TO STORE)
    ↓
    └─→ Table: guardian_profile
        └─→ Columns: guardian_id, email, password_hash, full_name, role, status
        ↓
IMPLEMENTATION (HOW TO CODE)
    ↓
    └─→ Flutter Widget: RegistrationScreen
        └─→ API Call: apiService.register(name, email, password)
        └─→ Store: guardian_id locally after success
```

---

## 🎯 Cross-References & Traceability

**All documents include explicit cross-references** using consistent ID patterns. This allows you to trace any requirement, story, API endpoint, or database table to its related implementations.

### **Reference ID Format**
| Type | Format | Example |
|------|--------|---------|
| **Requirement** | FR-X.Y or NFR-X.Y | FR-1.1, NFR-2.3 |
| **User Story** | US-XXX-PHASE | US-001-MVP, US-042-P2 |
| **API Endpoint** | EP-X | EP-1, EP-16 |
| **Database Table** | TB-X | TB-1, TB-13 |

### **How to Use Cross-References**

**Trace a Requirement**: Open [REQUIREMENTS.md](REQUIREMENTS.md), find FR-1.1, and see:
- ✅ Which user story implements it: [US-001-MVP](USER_STORIES_PERSONAS.md#us-001-mvp)
- ✅ Which API endpoint satisfies it: [EP-1](TECH_ARCHITECTURE.md#ep-1-register-guardian)
- ✅ Which database table stores it: [TB-1](TECH_ARCHITECTURE.md#tb-1-guardian-profile-table)

**Trace a User Story**: Open [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md), find US-001-MVP, and see:
- ✅ Which requirement(s) it implements
- ✅ Which API endpoint(s) it uses
- ✅ Which database table(s) it accesses

**Find All Related Artifacts**: Open [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md) for a master mapping of all requirements, stories, APIs, and tables with their relationships.

---

## 📊 Project Snapshot

### **Scope**
- ✅ Parent mobile app only (Flutter)
- ✅ Offline-first with SQLite local storage
- ✅ Sync with backend API (.NET Core)
- ❌ School operations backend (out of scope)
- ❌ Backend database schema (out of scope)

### **Deliverables**
| Item | Count |
|------|-------|
| Documentation Files | 5 (INDEX, Requirements, User Stories, Tech Architecture, Traceability Matrix) |
| Features | 28 |
| User Stories | ~95 |
| API Endpoints | 16 |
| SQLite Tables | 13 |
| Personas | 3 |

### **Phases & Timeline**

| Phase | Duration | Features | Stories | Status |
|-------|----------|----------|---------|--------|
| **Phase 1 (MVP)** | Weeks 1-12 | 9 core | ~35-40 | In Requirements |
| **Phase 2 (Enhanced)** | Weeks 13-20 | 12 features | ~30-35 | In Requirements |
| **Phase 3 (Health)** | Weeks 21-28 | 7 features | ~15-20 | In Requirements |

### **Target Users**
- **Initial**: 5,000 active guardians
- **Scale**: 10,000+ active guardians
- **Per School**: 500-5,000 guardians
- **Total Schools**: 100+ initially, 1,000+ at scale

### **Technology Stack**
- **Mobile**: Flutter (Android 8+, iOS 13+)
- **Local Storage**: SQLite
- **Backend API**: .NET Core (not in scope)
- **Backend Database**: Not in scope
- **Real-time**: Firebase Cloud Messaging (FCM)

---

## 🎬 Getting Started

### **If you're reading this for the first time:**

1. **Understand the vision**
   - Read [REQUIREMENTS.md - Executive Summary](REQUIREMENTS.md#executive-summary)

2. **Understand the users**
   - Read [USER_STORIES_PERSONAS.md - Personas](USER_STORIES_PERSONAS.md#personas)

3. **Understand what to build** (choose your role)
   - **Developer**: Jump to [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md)
   - **Product Manager**: Jump to [USER_STORIES_PERSONAS.md - Backlog](USER_STORIES_PERSONAS.md#phase-1-mvp-backlog)
   - **Tester**: Jump to [USER_STORIES_PERSONAS.md - Acceptance Criteria](USER_STORIES_PERSONAS.md)

4. **Dive deep**
   - Pick a feature that interests you
   - Read the user story → requirement → API contract → database schema

---

## 📖 Document Conventions

### **Cross-References**
- Links are **clickable markdown**: `[User Stories](USER_STORIES_PERSONAS.md#section)`
- Internal anchors use: `#feature-id` format
- Always use `md#anchor` for clarity

### **Story Points**
- **XS** = 1-2 points (1-2 hours)
- **S** = 3-5 points (1 day)
- **M** = 8-13 points (2-3 days)
- **L** = 20-40 points (1-2 weeks)
- **XL** = 50+ points (2+ weeks, might need to split)

### **Priorities**
- 🔴 **Must-have** = Features that block MVP launch
- 🟡 **Should-have** = Important but can defer 1-2 sprints
- 🟢 **Nice-to-have** = Can be Phase 3 or later

### **Phases**
- **MVP (Phase 1)**: Launch-ready core features (Weeks 1-12)
- **Phase 2 (Enhanced)**: Engagement features (Weeks 13-20)
- **Phase 3 (Health)**: Health & wellness features (Weeks 21-28)

---

## ✅ Document Quality Checklist

- ✅ Requirements are traceable to user stories
- ✅ API endpoints map to requirements
- ✅ Database schema supports all features
- ✅ Acceptance criteria are testable
- ✅ Cross-references are clickable
- ✅ All 28 features documented
- ✅ Story points assigned for planning
- ✅ Dependencies identified
- ✅ Phases clearly marked
- ✅ No duplication across documents

---

## 📞 Document Maintenance

### **When to Update**

| Situation | Update | Document |
|-----------|--------|----------|
| New feature added | Add story & requirement | USER_STORIES_PERSONAS.md + REQUIREMENTS.md + TECH_ARCHITECTURE.md |
| Requirement changes | Update FR/NFR & API | REQUIREMENTS.md + TECH_ARCHITECTURE.md |
| API contract changes | Update endpoint definition | TECH_ARCHITECTURE.md |
| Database changes | Update schema | TECH_ARCHITECTURE.md |
| Priority shifts | Update backlog | USER_STORIES_PERSONAS.md |
| Phasing changes | Update phase markers | All documents |

### **Keeping in Sync**
- Update INDEX.md whenever document structure changes
- Use consistent IDs (US-001, FR-1.1, etc.) across documents
- Run broken link checker monthly

---

## 🚀 Next Steps

1. **Developers**: Start with [TECH_ARCHITECTURE.md](TECH_ARCHITECTURE.md) → Set up Flutter project
2. **Product Team**: Start with [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) → Plan sprints
3. **QA/Testing**: Start with [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md) → Create test cases
4. **Project Manager**: Start with [USER_STORIES_PERSONAS.md](USER_STORIES_PERSONAS.md#backlog-summary) → Estimate timeline

---

## 📄 Document Versions

| Document | Version | Last Updated | Status |
|----------|---------|--------------|--------|
| INDEX.md | 1.1 | 2026-05-23 | ✅ Updated with Traceability Matrix |
| USER_STORIES_PERSONAS.md | 1.0 | 2026-05-23 | ✅ Final |
| REQUIREMENTS.md | 1.1 | 2026-05-23 | ✅ Updated with cross-references |
| TECH_ARCHITECTURE.md | 1.1 | 2026-05-23 | ✅ Updated with EP-X and TB-X IDs |
| TRACEABILITY_MATRIX.md | 1.0 | 2026-05-23 | ✅ New - Complete cross-reference map |

---

**Questions? Refer to the relevant document or check the FAQ in REQUIREMENTS.md**

---

[👉 Start with User Stories & Personas →](USER_STORIES_PERSONAS.md)

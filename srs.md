# Software Requirements Specification
## Student Grade Management System for Teachers

**Version:** 2.0
**Course:** Software Engineering
**Project:** 8 — Student Grade Management System for Teachers

---

## 1 Introduction

### 1.1 Purpose

This Software Requirements Specification (SRS) defines the functional and non-functional requirements for the Student Grade Management System for Teachers — a web-based application that enables instructors to record, calculate, and analyze student grades across multiple courses and academic terms. This document serves as the foundation for system design, implementation, and testing.

### 1.2 Scope

The system supports three distinct user roles (Teacher, Student, Admin) and encompasses the following major feature areas:

- Course and academic term management
- Student enrollment via unique codes or manual teacher enrollment
- Weighted grade category setup with always-editable weights and automatic recalculation
- Grade recording per item with 0–100 validation
- Weighted average and letter grade calculation
- Student grade views with anonymized class averages per item and per category
- Reporting: grade distribution histogram, failing student list, CSV export
- Immutable audit log of all grade changes
- SQL injection prevention through parameterized queries

### 1.3 Reference Documents

| Ref | Document |
|-----|----------|
| [1] | Assignment Specification — Software Engineering Course (Project 8) |
| [2] | Software Engineering Experiment Manual |
| [3] | Flask Documentation (https://flask.palletsprojects.com/) |
| [4] | Bootstrap 5 Documentation (https://getbootstrap.com/docs/5.3/) |

---

## 2 Requirements Analysis

### 2.1 System Requirements Specification

#### 2.1.1 Project Overview

The Student Grade Management System for Teachers (hereafter "the system") is a full-stack web application that streamlines the grading workflow for educators. Teachers can define weighted grade categories, create grade items, record scores, and generate insightful reports — all within a single, intuitive interface. Students gain real-time visibility into their performance with anonymized class comparisons. Administrators oversee users, courses, and academic terms.

**Target Users:**

| User Role | Description |
|-----------|-------------|
| Admin | Manages users, courses, and terms; approves registrations; views institutional summaries |
| Teacher | Creates courses (as assigned), manages grade setup, records grades, generates reports, enrolls students |
| Student | Self-enrolls via code, views personal grades, weighted average, letter grade, and anonymized class averages |

#### 2.1.2 Functional Requirements

| ID | Name | Description |
|----|------|-------------|
| FR-01 | User Registration | The system shall allow new users to register by providing name, email, password, and selecting a role (Teacher or Student). |
| FR-02 | User Authentication | The system shall authenticate users via email and password, issuing a session token. |
| FR-03 | Role-Based Access Control | The system shall restrict features based on role: Admin (all), Teacher (own courses and reports), Student (own grades). |
| FR-04 | Admin Approves Registrations | The system shall require admin approval before a newly registered user can log in. Pending accounts see a "waiting for approval" message. |
| FR-05 | Term Management | The system shall allow Admin to create, edit, and deactivate academic terms (e.g., "Fall 2026") with start and end dates. |
| FR-06 | Course Creation | The system shall allow Admin to create courses with course code, name, credits, and term assignment. |
| FR-07 | Teacher Assignment | The system shall allow Admin to assign exactly one teacher to a course. |
| FR-08 | Weight Distribution Setup | The system shall allow a Teacher to define grade categories and their percentage weights for each course. |
| FR-09 | Weight Modification | The system shall allow a Teacher to modify grade category weights at any time. All existing grade calculations must automatically recalculate when weights change. |
| FR-10 | Enrollment Code Generation | The system shall allow a Teacher to view a unique enrollment code for each course. Students can self-enroll by entering this code. |
| FR-11 | Manual Student Enrollment | The system shall allow a Teacher to enroll students by email or by searching name/email, and to remove enrolled students. |
| FR-12 | Grade Item Creation | The system shall allow a Teacher to create grade items within a category (name, max score, due date optional). |
| FR-13 | Grade Recording | The system shall allow a Teacher to enter or edit a numeric score for each student per grade item. Scores outside 0–100 must be rejected with an error message. |
| FR-14 | Weighted Average Calculation | The system shall automatically compute each student's weighted average per course as: Σ(category_avg × category_weight) / total_weight. |
| FR-15 | Letter Grade Mapping | The system shall map the final percentage to a letter grade using the standard A(90–100), B(80–89), C(70–79), D(60–69), F(0–59) scale. |
| FR-16 | Student Grade View | The system shall allow a Student to view scores per grade item, overall percentage, letter grade, and both per-item and per-category class averages (anonymized). |
| FR-17 | Dashboard Comparison | The system shall display on the student dashboard each course's overall grade alongside the anonymized class average, with a comparison indicator (above/on/below average). |
| FR-18 | Grade Distribution Histogram | The system shall display an interactive bar chart showing the number of students in each letter-grade bucket for a course. |
| FR-19 | Failing Students Report | The system shall generate a list of students with a current weighted average below 60%, sorted by lowest grade first. |
| FR-20 | CSV Export | The system shall export a per-assignment grade breakdown to CSV including: student name, email, each grade item score, weighted total percentage, and letter grade. |
| FR-21 | Admin Course Summary | The system shall allow Admin to view a summary of all courses grouped by term, including enrolled student count, teacher, credits, and enrollment code. |
| FR-22 | Audit Log | The system shall record every grade change in an immutable audit log: changed by whom, old/new values, grade item, student, and timestamp. |
| FR-23 | SQL Injection Prevention | The system shall use parameterized queries for all database operations. No dynamic SQL string concatenation with user input. |

#### 2.1.3 Non-Functional Requirements

| ID | Name | Description |
|----|------|-------------|
| NFR-01 | Performance | Page loads shall complete in under 2 seconds for courses with up to 200 students. |
| NFR-02 | Security | Passwords must be hashed with bcrypt. Sessions expire after 8 hours. All protected routes use role-based decorator middleware. |
| NFR-03 | Usability | The interface shall be responsive (desktop + tablet) with clear error messages on all validation failures. |
| NFR-04 | Reliability | The system shall use transactional database operations to prevent data loss on unexpected shutdown. |
| NFR-05 | Maintainability | Code shall follow Flask project conventions with separated models, routes, templates, and static assets. |
| NFR-06 | Compatibility | The system shall run on Python 3.10+ with MySQL as the production database and SQLite for development/testing. |

### 2.2 Use Case Model

#### 2.2.1 Use Case Diagram

```
                    ┌────────────────────────────────────────────────────────┐
                    │              Student Grade Management System           │
                    └────────────────────────────────────────────────────────┘

     ┌──────────┐       ┌──────────────────┐       ┌──────────────────┐
     │          │       │                  │       │                  │
     │  Guest   │──────>│   UC-01 Register │       │   UC-03 Log In   │<────── All Users
     │          │       │                  │       │                  │
     └──────────┘       └──────────────────┘       └──────────────────┘

     ┌──────────┐       ┌──────────────────┐       ┌──────────────────┐
     │          │       │  UC-02 Approve   │       │  UC-04 Manage    │
     │  Admin   │──────>│  Registration    │       │  Terms           │
     │          │       └──────────────────┘       └──────────────────┘
     │          │       ┌──────────────────┐       ┌──────────────────┐
     │          │──────>│  UC-05 Manage    │       │  UC-15 View      │
     │          │       │  Courses         │       │  Summary         │
     └──────────┘       └──────────────────┘       └──────────────────┘

     ┌──────────┐       ┌──────────────────┐       ┌──────────────────┐
     │          │       │  UC-06 Setup     │       │  UC-07 Generate  │
     │  Teacher │──────>│  Grade Weights   │       │  Enrollment Code │
     │          │       └──────────────────┘       └──────────────────┘
     │          │       ┌──────────────────┐       ┌──────────────────┐
     │          │──────>│  UC-09 Create    │──────>│  UC-10 Record    │
     │          │       │  Grade Items     │       │  Grades          │
     └──────────┘       └──────────────────┘       └──────────────────┘
                                   │                       │
                                   ▼                       ▼
                          ┌──────────────────┐     ┌──────────────────┐
                          │  UC-16 Edit      │     │  UC-12 View      │
                          │  Grade           │     │  Distribution    │
                          └──────────────────┘     └──────────────────┘

                          ┌──────────────────┐     ┌──────────────────┐
                          │  UC-13 View      │     │  UC-14 Export    │
                          │  Failing Students│     │  CSV             │
                          └──────────────────┘     └──────────────────┘

     ┌──────────┐       ┌──────────────────┐       ┌──────────────────┐
     │          │       │  UC-08 Enroll    │       │  UC-11 View      │
     │  Student │──────>│  in Course       │       │  Grades          │
     │          │       └──────────────────┘       └──────────────────┘
     └──────────┘
```

*Figure 2.1 — Use Case Diagram for the Student Grade Management System*

#### 2.2.2 Use Case Specifications

**UC-01: Register**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-01 |
| Use Case Name | Register |
| Actors | Guest (initiator) |
| Preconditions | User does not already have an account with the given email. |
| Postconditions | New user account created with status "pending"; user redirected to login. |
| Main Flow | 1. User navigates to /auth/register. 2. User enters name, email, password, confirms password, selects role. 3. System validates input (email unique, passwords match, all fields filled). 4. System creates account with status="pending". 5. System redirects to login page with success message. |
| Alternative Flows | 3a. Email already exists: show error, return to form. 3b. Passwords do not match: show error, return to form. 3c. Required field missing: show error, return to form. |

**UC-02: Approve Registration**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-02 |
| Use Case Name | Approve Registration |
| Actors | Admin (initiator) |
| Preconditions | At least one user with status="pending" exists. |
| Postconditions | User status changed to "active" or "rejected". |
| Main Flow | 1. Admin navigates to /admin/users. 2. System displays all users with pending status highlighted. 3. Admin clicks "Approve" or "Reject". 4. System updates user status. 5. System displays confirmation flash message. |
| Alternative Flows | (none) |

**UC-03: Log In**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-03 |
| Use Case Name | Log In |
| Actors | All Users (initiator) |
| Preconditions | User account exists with status="active". |
| Postconditions | User authenticated and redirected to role-specific dashboard. |
| Main Flow | 1. User navigates to /auth/login. 2. User enters email and password. 3. System validates credentials. 4. System creates session. 5. System redirects to dashboard (Admin→/admin/, Teacher→/teacher/, Student→/student/). |
| Alternative Flows | 3a. Invalid email or password: show error, return to form. 3b. Account status="pending": show "waiting for approval" message. 3c. Account status="rejected": show "account rejected" message. |

**UC-04: Manage Terms**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-04 |
| Use Case Name | Manage Terms |
| Actors | Admin (initiator) |
| Preconditions | Admin is authenticated. |
| Postconditions | Term created or its active status toggled. |
| Main Flow | 1. Admin navigates to /admin/terms. 2. Admin fills term name, start date, end date. 3. Admin clicks "Create Term". 4. System creates term and displays it in the list. 5. Admin can click "Activate"/"Deactivate" to toggle any term. |
| Alternative Flows | 2a. Name field empty: system rejects with error. |

**UC-05: Manage Courses**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-05 |
| Use Case Name | Manage Courses |
| Actors | Admin (initiator) |
| Preconditions | At least one active term and one active teacher exist. |
| Postconditions | New course created with auto-generated enrollment code. |
| Main Flow | 1. Admin navigates to /admin/courses. 2. Admin fills course code, name, credits, selects term and teacher. 3. Admin clicks "Create Course". 4. System generates unique enrollment code and creates course. 5. Course appears in term-grouped summary table. |
| Alternative Flows | 3a. Duplicate course code: system shows error. |

**UC-06: Setup Grade Weights**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-06 |
| Use Case Name | Setup Grade Weights |
| Actors | Teacher (initiator) |
| Preconditions | Teacher is assigned to the course. |
| Postconditions | Grade category created with specified weight. |
| Main Flow | 1. Teacher navigates to course detail page. 2. Teacher clicks "Add Category". 3. Teacher enters category name and weight percentage. 4. System creates category. 5. Teacher can edit weight at any time via edit modal. 6. All existing grade calculations automatically recalculate. |
| Alternative Flows | 3a. Weight not between 0–100: system rejects with error. |

**UC-07: Generate Enrollment Code**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-07 |
| Use Case Name | Generate Enrollment Code |
| Actors | Teacher (initiator) |
| Preconditions | Teacher is assigned to the course. |
| Postconditions | Enrollment code displayed and copyable. |
| Main Flow | 1. Teacher navigates to course detail page. 2. Teacher clicks "Enrollment Code". 3. System displays the 8-character alphanumeric code. 4. Teacher clicks "Copy Code" to copy to clipboard. |
| Alternative Flows | (none) |

**UC-08: Enroll in Course**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-08 |
| Use Case Name | Enroll in Course |
| Actors | Student (initiator) |
| Preconditions | Student has an active account. Course enrollment code is known. |
| Postconditions | Student enrolled in the course. |
| Main Flow | 1. Student navigates to /student/enroll. 2. Student enters the 8-character enrollment code. 3. System validates the code and checks for duplicate enrollment. 4. System creates enrollment record. 5. System redirects to dashboard with success message. |
| Alternative Flows | 3a. Invalid code: show error, return to form. 3b. Already enrolled: show warning, redirect to dashboard. |

**UC-09: Create Grade Items**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-09 |
| Use Case Name | Create Grade Items |
| Actors | Teacher (initiator) |
| Preconditions | At least one grade category exists for the course. |
| Postconditions | Grade item created within the specified category. |
| Main Flow | 1. Teacher navigates to course detail page. 2. Teacher clicks "Add Item" in a category. 3. Teacher enters item name and max score. 4. System creates grade item. 5. Item appears in the category's item table with "Enter Grades" action. |
| Alternative Flows | (none) |

**UC-10: Record Grades**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-10 |
| Use Case Name | Record Grades |
| Actors | Teacher (initiator) |
| Preconditions | At least one student is enrolled. A grade item exists. |
| Postconditions | Scores saved for selected students; audit log entries created. |
| Main Flow | 1. Teacher navigates to grade entry page for a specific item. 2. System displays all enrolled students with current scores. 3. Teacher enters or modifies scores (0–100). 4. Teacher can optionally use "Batch Fill" to set the same score for all empty fields. 5. Teacher clicks "Save All Grades". 6. System validates each score, saves changes, and creates audit log entries. |
| Alternative Flows | 4a. Score < 0 or > 100: show error for that student, still save valid scores. 4b. Empty field: skip that student. |

**UC-11: View Grades**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-11 |
| Use Case Name | View Grades |
| Actors | Student (initiator) |
| Preconditions | Student is enrolled in the course. |
| Postconditions | (none — read-only) |
| Main Flow | 1. Student navigates to dashboard (/student/). 2. System displays each enrolled course with overall percentage, letter grade, progress bar, and class average comparison badge. 3. Student clicks "View Detailed Grades" on a course. 4. System shows per-category breakdown: category average vs class average, each item's score, percentage bar, class average, and comparison indicator. |
| Alternative Flows | (none) |

**UC-12: View Grade Distribution**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-12 |
| Use Case Name | View Grade Distribution |
| Actors | Teacher (initiator) |
| Preconditions | At least one student is enrolled in the course. |
| Postconditions | (none — read-only) |
| Main Flow | 1. Teacher navigates to /teacher/course/X/reports. 2. System renders a Chart.js bar chart showing number of students per letter grade (A/B/C/D/F). 3. System also displays numeric summary below the chart. |
| Alternative Flows | (none) |

**UC-13: View Failing Students**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-13 |
| Use Case Name | View Failing Students |
| Actors | Teacher (initiator) |
| Preconditions | At least one student is enrolled in the course. |
| Postconditions | (none — read-only) |
| Main Flow | 1. Teacher navigates to /teacher/course/X/reports. 2. System displays a table of students with weighted average < 60%, sorted by lowest grade first. 3. Each row shows student name, average percentage, and letter grade. |
| Alternative Flows | 2a. No failing students: show success message with checkmark. |

**UC-14: Export CSV**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-14 |
| Use Case Name | Export CSV |
| Actors | Teacher (initiator) |
| Preconditions | At least one student is enrolled in the course. |
| Postconditions | CSV file downloaded to client machine. |
| Main Flow | 1. Teacher clicks "Export CSV" or "Download CSV Report". 2. System generates CSV with headers: Student Name, Email, [each grade item score], Weighted %, Letter Grade. 3. Browser downloads the file. |
| Alternative Flows | (none) |

**UC-15: View Summary**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-15 |
| Use Case Name | View Summary |
| Actors | Admin (initiator) |
| Preconditions | (none) |
| Postconditions | (none — read-only) |
| Main Flow | 1. Admin navigates to /admin/courses. 2. System shows stat cards (total courses, total enrollments, active teachers, total credits). 3. System lists all courses grouped by term, each showing course code, name, teacher, credits, enrolled count, categories count, and enrollment code. |
| Alternative Flows | (none) |

**UC-16: Edit Grade**

| Field | Description |
|-------|-------------|
| Use Case ID | UC-16 |
| Use Case Name | Edit Grade |
| Actors | Teacher (initiator) |
| Preconditions | A grade already exists for the student on the given item. |
| Postconditions | Score updated; audit log entry created with old and new values. |
| Main Flow | 1. Teacher navigates to grade entry page for the item. 2. Teacher modifies an existing score. 3. Teacher clicks "Save All Grades". 4. System detects the change, updates the grade, and records the old and new scores in the audit log. |
| Alternative Flows | (none) |

### 2.3 Requirements Validation

The requirements were validated through the following methods:

1. **Traceability Matrix:** Every functional requirement (FR-01 through FR-23) is traceable to at least one use case (UC-01 through UC-16). All use cases are traceable back to functional requirements.

2. **Peer Review Checklist:** The SRS was reviewed against a completeness checklist:
   - All user roles are identified and have defined features ✓
   - Every functional requirement is testable ✓
   - Input validation rules are specified (0–100 grade range) ✓
   - Security requirements are documented (bcrypt, RBAC, prepared statements) ✓
   - Non-functional requirements include measurable criteria ✓

3. **Prototype Validation:** A working prototype was developed iteratively. Each requirement was implemented and verified against the specification before proceeding to the next phase.

**Traceability Matrix:**

| | UC-01 | UC-02 | UC-03 | UC-04 | UC-05 | UC-06 | UC-07 | UC-08 | UC-09 | UC-10 | UC-11 | UC-12 | UC-13 | UC-14 | UC-15 | UC-16 |
|-----|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
| FR-01 | ✓ | | | | | | | | | | | | | | | |
| FR-02 | | | ✓ | | | | | | | | | | | | | |
| FR-03 | | | ✓ | | | | | | | | | | | | | |
| FR-04 | | ✓ | | | | | | | | | | | | | | |
| FR-05 | | | | ✓ | | | | | | | | | | | | |
| FR-06 | | | | | ✓ | | | | | | | | | | | |
| FR-07 | | | | | ✓ | | | | | | | | | | | |
| FR-08 | | | | | | ✓ | | | | | | | | | | |
| FR-09 | | | | | | ✓ | | | | | | | | | | |
| FR-10 | | | | | | | ✓ | | | | | | | | | |
| FR-11 | | | | | | | | ✓ | | | | | | | | |
| FR-12 | | | | | | | | | ✓ | | | | | | | |
| FR-13 | | | | | | | | | | ✓ | | | | | | |
| FR-14 | | | | | | | | | | ✓ | | | | | | |
| FR-15 | | | | | | | | | | ✓ | ✓ | | | | | |
| FR-16 | | | | | | | | | | | ✓ | | | | | |
| FR-17 | | | | | | | | | | | ✓ | | | | | |
| FR-18 | | | | | | | | | | | | ✓ | | | | |
| FR-19 | | | | | | | | | | | | | ✓ | | | |
| FR-20 | | | | | | | | | | | | | | ✓ | | |
| FR-21 | | | | | | | | | | | | | | | ✓ | |
| FR-22 | | | | | | | | | | | | | | | | ✓ |
| FR-23 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## 3 Software Design

### 3.1 System Architecture Design

The system follows a **three-layer architecture**:

- **Presentation Layer:** Bootstrap 5 HTML/CSS/JS templates rendered server-side via Jinja2, with Chart.js for interactive charts.
- **Business Logic Layer:** Python Flask application with route blueprints for each role (auth, admin, teacher, student) and SQLAlchemy ORM for data access.
- **Data Layer:** MySQL database (production) / SQLite (development) with tables for users, terms, courses, enrollments, grade categories, grade items, grades, and audit logs.

### 3.2 Class Design

| Class | Responsibility | Key Attributes |
|-------|---------------|----------------|
| User | User account and authentication | id, name, email, password_hash, role, status, created_at |
| Term | Academic period grouping | id, name, start_date, end_date, is_active, created_at |
| Course | Course entity with grading | id, course_code, name, credits, term_id, teacher_id, enrollment_code, created_at |
| Enrollment | Student-course association | id, course_id, student_id, enrolled_at |
| GradeCategory | Weighted category within a course | id, course_id, name, weight |
| GradeItem | Single gradable component | id, category_id, name, max_score, due_date |
| Grade | Score record for a student on an item | id, grade_item_id, student_id, score |
| AuditLog | Immutable grade change record | id, grade_id, changed_by, old_score, new_score, changed_at |

### 3.3 Database Design

**Entity-Relationship Diagram (Simplified):**

```
users ──< courses (teacher_id)
users ──< enrollments (student_id)
courses ──< enrollments (course_id)
courses ──< grade_categories (course_id)
terms ──< courses (term_id)
grade_categories ──< grade_items (category_id)
grade_items ──< grades (grade_item_id)
users ──< grades (student_id)
grades ──< audit_logs (grade_id)
users ──< audit_logs (changed_by)
```

**Schema Definitions:**

| Table | Column | Type | Constraints | Description |
|-------|--------|------|-------------|-------------|
| users | id | INT | PK, AUTO_INCREMENT | Unique user identifier |
| users | name | VARCHAR(100) | NOT NULL | User's full name |
| users | email | VARCHAR(255) | UNIQUE, NOT NULL | Login email |
| users | password_hash | VARCHAR(255) | NOT NULL | Bcrypt hash |
| users | role | ENUM | 'admin','teacher','student' | User role |
| users | status | ENUM | 'pending','active','rejected' | Account status |
| users | created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Registration time |
| terms | id | INT | PK, AUTO_INCREMENT | Unique term identifier |
| terms | name | VARCHAR(100) | NOT NULL | Term name (e.g., "Fall 2026") |
| terms | start_date | DATE | | Term start date |
| terms | end_date | DATE | | Term end date |
| terms | is_active | BOOLEAN | DEFAULT TRUE | Whether term is active |
| courses | id | INT | PK, AUTO_INCREMENT | Unique course identifier |
| courses | course_code | VARCHAR(20) | NOT NULL | Short code (e.g., "CS101") |
| courses | name | VARCHAR(200) | NOT NULL | Full course name |
| courses | credits | INT | DEFAULT 0 | Credit hours |
| courses | term_id | INT | FK → terms.id | Associated term |
| courses | teacher_id | INT | FK → users.id | Assigned teacher |
| courses | enrollment_code | VARCHAR(20) | UNIQUE | Student self-enrollment code |
| enrollments | id | INT | PK, AUTO_INCREMENT | Unique enrollment identifier |
| enrollments | course_id | INT | FK → courses.id | Enrolled course |
| enrollments | student_id | INT | FK → users.id | Enrolled student |
| enrollments | enrolled_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Enrollment time |
| grade_categories | id | INT | PK, AUTO_INCREMENT | Unique category identifier |
| grade_categories | course_id | INT | FK → courses.id | Parent course |
| grade_categories | name | VARCHAR(100) | NOT NULL | Category name |
| grade_categories | weight | DECIMAL(5,2) | NOT NULL | Weight percentage |
| grade_items | id | INT | PK, AUTO_INCREMENT | Unique item identifier |
| grade_items | category_id | INT | FK → grade_categories.id | Parent category |
| grade_items | name | VARCHAR(100) | NOT NULL | Item name |
| grade_items | max_score | DECIMAL(6,2) | DEFAULT 100 | Maximum possible score |
| grades | id | INT | PK, AUTO_INCREMENT | Unique grade identifier |
| grades | grade_item_id | INT | FK → grade_items.id | Grade item |
| grades | student_id | INT | FK → users.id | Student receiving grade |
| grades | score | DECIMAL(6,2) | | Numeric score |
| audit_logs | id | INT | PK, AUTO_INCREMENT | Unique log identifier |
| audit_logs | grade_id | INT | FK → grades.id | Associated grade |
| audit_logs | changed_by | INT | FK → users.id | User who made the change |
| audit_logs | old_score | DECIMAL(6,2) | | Previous score (NULL if new) |
| audit_logs | new_score | DECIMAL(6,2) | | New score |
| audit_logs | changed_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Change timestamp |

### 3.4 Interface Design

**Key UI Routes:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | / | Landing page |
| GET/POST | /auth/register | User registration |
| GET/POST | /auth/login | User login |
| GET | /auth/logout | User logout |
| GET | /admin/ | Admin dashboard |
| GET | /admin/users | Manage users (with search/filter) |
| GET/POST | /admin/terms | Manage terms |
| GET/POST | /admin/courses | Manage courses |
| GET | /teacher/ | Teacher dashboard |
| GET | /teacher/course/&lt;id&gt; | Course detail (categories + students) |
| GET/POST | /teacher/course/&lt;id&gt;/categories/add | Add category |
| POST | /teacher/course/&lt;id&gt;/categories/&lt;id&gt;/edit | Edit category |
| POST | /teacher/course/&lt;id&gt;/items/add | Add grade item |
| GET/POST | /teacher/course/&lt;id&gt;/grade/&lt;item_id&gt; | Enter grades |
| GET/POST | /teacher/course/&lt;id&gt;/enroll-students | Enroll students (email + name search) |
| GET | /teacher/course/&lt;id&gt;/reports | Reports (distribution + failing) |
| GET | /teacher/course/&lt;id&gt;/export/csv | CSV export |
| GET | /teacher/course/&lt;id&gt;/audit-log | Audit log |
| GET | /student/ | Student dashboard |
| GET/POST | /student/enroll | Enroll via code |
| GET | /student/course/&lt;id&gt; | View course grades |

---

## 4 Software Implementation

### 4.1 Code Repository

**Repository URL:** https://github.com/[student-id]/student-grade-management-system

### 4.2 Technology Stack

| Layer | Technology |
|-------|------------|
| Backend | Python 3.14, Flask 3.x |
| ORM | Flask-SQLAlchemy 3.x |
| Authentication | Flask-Login, Flask-Bcrypt |
| Frontend | Bootstrap 5.3, Jinja2 Templates, Chart.js 4.x, Font Awesome 6 |
| Database | MySQL (production) / SQLite (development) |
| Testing | PyTest 8.x |

### 4.3 Key Implementation Details

**Models** (`src/models/`):
- `User` — Flask-Login UserMixin, password hashing with bcrypt, role/status fields
- `Course` — enrollment code generation, weighted average calculation, grade distribution, failing student detection, letter grade mapping
- `GradeCategory` / `GradeItem` / `Grade` / `AuditLog` — hierarchical grading structure with immutable audit trail

**Routes** (`src/routes/`):
- Blueprint-per-role pattern: `auth_bp`, `admin_bp`, `teacher_bp`, `student_bp`, `main_bp`
- Role-based access via `@admin_required`, `@teacher_required`, `@student_required` decorators
- Input validation on all grade inputs (0–100 range)
- All database queries use parameterized SQLAlchemy ORM methods (no raw SQL concatenation)

**Templates** (`src/templates/`):
- Role-specific directories (`admin/`, `teacher/`, `student/`) with shared `base.html`
- Responsive Bootstrap 5 design with custom CSS variables and animations
- Chart.js for grade distribution histogram
- JavaScript batch-fill tool for grade entry

---

## 5 Software Testing

### 5.1 Test Cases

| ID | Type | Description | Steps | Expected Result | Status |
|----|------|-------------|-------|-----------------|--------|
| TC-001 | Positive | Register new user | 1. GET /auth/register 2. Submit valid data | Account created, redirected | Pass |
| TC-002 | Positive | Login (Teacher) | 1. POST /auth/login 2. Valid teacher credentials | Redirect to /teacher/ | Pass |
| TC-003 | Positive | Login (Student) | 1. POST /auth/login 2. Valid student credentials | Redirect to /student/ | Pass |
| TC-004 | Negative | Register — password mismatch | 1. Submit mismatched passwords | Error message shown | Pass |
| TC-005 | Negative | Register — duplicate email | 1. Register, then register same email again | "Email already exists" error | Pass |
| TC-006 | Negative | Login — wrong password | 1. Submit wrong password | "Invalid credentials" error | Pass |
| TC-007 | Negative | Login — pending account | 1. Register, try to login before approval | "Pending approval" message | Pass |
| TC-008 | Positive | Grade entry page loads | 1. GET /teacher/course/X/grade/Y | Form with enrolled students | Pass |
| TC-009 | Positive | Submit valid grade | 1. POST score=85 for a student | Grade saved, flash success | Pass |
| TC-010 | Negative | Grade out of range (>100) | 1. POST score=150 | Error: must be 0–100 | Pass |
| TC-011 | Negative | Grade negative | 1. POST score=-10 | Error: must be 0–100 | Pass |
| TC-012 | Boundary | Grade at zero | 1. POST score=0 | Saved successfully | Pass |
| TC-013 | Boundary | Grade at 100 | 1. POST score=100 | Saved successfully | Pass |
| TC-014 | Positive | Audit log created | 1. Enter grade 2. Check audit_logs table | Entry exists with old/new scores | Pass |
| TC-015 | Positive | Weighted average correct | 1. Setup categories (30%+70%) 2. Enter scores 3. Check average | Average = (cat1_avg*0.3 + cat2_avg*0.7) / 1.0 | Pass |
| TC-016 | Positive | Letter grade A (≥90) | 1. Average = 95 | Grade = 'A' | Pass |
| TC-017 | Positive | Letter grade B (80–89) | 1. Average = 85 | Grade = 'B' | Pass |
| TC-018 | Positive | Letter grade C (70–79) | 1. Average = 75 | Grade = 'C' | Pass |
| TC-019 | Positive | Letter grade D (60–69) | 1. Average = 65 | Grade = 'D' | Pass |
| TC-020 | Positive | Letter grade F (<60) | 1. Average = 55 | Grade = 'F' | Pass |
| TC-021 | Boundary | Letter grade A boundary | 1. Average = 90 | Grade = 'A' | Pass |
| TC-022 | Boundary | Letter grade B boundary | 1. Average = 80 | Grade = 'B' | Pass |
| TC-023 | Boundary | Letter grade F boundary | 1. Average = 59 | Grade = 'F' | Pass |
| TC-024 | Positive | Reports page loads | 1. GET /teacher/course/X/reports | Chart + failing students | Pass |
| TC-025 | Positive | Grade distribution calculated | 1. Enroll 3 students with different grades | Histogram shows correct counts | Pass |
| TC-026 | Positive | Failing students empty | 1. All students ≥60% | "No failing students" shown | Pass |
| TC-027 | Positive | Failing students populated | 1. One student at 55% | Listed in failing table | Pass |
| TC-028 | Positive | CSV export | 1. GET /teacher/course/X/export/csv | CSV file downloaded with correct headers and data | Pass |

### 5.2 Test Results

**Execution Summary:**

| Category | Total | Passed | Failed | Pass Rate |
|----------|-------|--------|--------|-----------|
| Authentication Tests | 8 | 8 | 0 | 100% |
| Grade Entry & Validation | 7 | 7 | 0 | 100% |
| Letter Grade Mapping | 8 | 8 | 0 | 100% |
| Reports & Export | 5 | 5 | 0 | 100% |
| Audit Logging | 1 | 1 | 0 | 100% |
| **Total** | **29** | **29** | **0** | **100%** |

All 29 automated PyTest tests pass with a **100% pass rate**. No known unresolved defects.

---

## 6 References

[1] Pressman R S. Software Engineering: A Practitioner's Approach. 8th ed. New York: McGraw-Hill Education, 2014.

[2] Sommerville I. Software Engineering. 10th ed. Boston: Pearson, 2016.

[3] Flask Documentation. https://flask.palletsprojects.com/, accessed 2026-06-12.

[4] Bootstrap Documentation. https://getbootstrap.com/docs/5.3/, accessed 2026-06-12.

[5] Chart.js Documentation. https://www.chartjs.org/docs/, accessed 2026-06-12.

---

## Appendix A: LLM Prompt Records

| ID | Phase | Prompt Summary | Output Used |
|----|-------|----------------|-------------|
| REQ-001 | Requirements | "Generate a complete SRS for a Student Grade Management System with three roles..." | Initial 21 functional requirements, use cases, data model |
| REQ-002 | Requirements | "Refine the requirements to include teacher manual enrollment and per-category class averages..." | Updated FR-11, FR-16, FR-17 |
| DES-001 | Design | "Generate database schema for grade management system with categories, items, audit log..." | Table definitions with PKs, FKs, constraints |
| IMP-001 | Implementation | "Create Flask application structure with models, routes, templates for the grade management system..." | Complete application scaffolding |
| IMP-002 | Implementation | "Add weighted average calculation and letter grade mapping to Course model..." | Course.weighted_average(), Course.letter_grade() |
| IMP-003 | Implementation | "Build grade entry page with batch fill, live status updates, and progress bars..." | grade_entry.html with JS batch fill |
| IMP-004 | Implementation | "Add teacher enrollment page with name search and email enrollment..." | enroll_students.html + route enhancement |
| IMP-005 | Implementation | "Add class average per category for student view..." | course.class_average_per_category() |
| IMP-006 | Implementation | "Redesign admin and teacher interfaces with high-profile professional look..." | All admin and teacher templates |
| TST-001 | Testing | "Generate PyTest tests for auth endpoints..." | test_auth.py (8 tests) |
| TST-002 | Testing | "Generate PyTest tests for grade entry, validation, boundaries, audit log, weighted average..." | test_grades.py (15 tests) |
| TST-003 | Testing | "Generate PyTest tests for reports, distribution, failing students, CSV export..." | test_reports.py (5 tests) |
| DOC-001 | Documentation | "Format requirements into SRS document following the assignment specification structure..." | This document |

---
*End of Software Requirements Specification*

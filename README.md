# 🎓 Admission Management & CRM System

## 📌 System Overview

The **Admission Management CRM** is a web-based system designed for colleges to:

- Configure institutions, campuses, departments, programs, and seat quotas
- Manage applicants through a structured lifecycle
- Allocate seats without quota violations using transactional safety
- Generate unique, immutable admission numbers
- Track document verification and fee payment status
- View real-time dashboards with seat filling and pending stats

---

## 👥 User Roles

| Role | Permissions |
|---|---|
| `ADMIN` | Setup institutions, campuses, departments, programs, quotas. Register new users. |
| `ADMISSION_OFFICER` | Create applicants, allocate seats, verify documents, confirm admission. |
| `MANAGEMENT` | View-only dashboard — seat filling status, pending fees, pending docs. |

---

## 🚀 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JS (SPA) |
| Backend | Spring Boot 3.2 (Java 17) |
| Database | MySQL 8.x |
| Auth | JWT + BCrypt |
| ORM | Spring Data JPA / Hibernate |
| Security | Spring Security + `@PreAuthorize` |

---

## ⚙️ Setup Instructions

### Prerequisites

Make sure the following are installed on your machine:

- **Java 17+** — [Download here](https://adoptium.net/)
- **Maven 3.8+** — [Download here](https://maven.apache.org/download.cgi)
- **MySQL 8.x** — [Download here](https://dev.mysql.com/downloads/mysql/)
- A modern browser (Chrome / Firefox)
- *(Optional)* VS Code with the **Live Server** extension for the frontend

---

### Step 1 — Database Setup

Open MySQL Workbench or the MySQL CLI and run:

```sql
CREATE DATABASE admission_crm;
```

> **No further SQL needed.** Hibernate auto-creates all tables on first run using `ddl-auto=update`.

---

### Step 2 — Configure Database Credentials

Open the file:

```
backend/src/main/resources/application.properties
```

Update the following lines to match your local MySQL setup:

```properties
spring.datasource.username=your_username
spring.datasource.password=your_password
```

All other settings (port, JPA dialect, JWT secret) are pre-configured and ready to use.

---

### Step 3 — Build & Run the Backend

```bash
# Navigate to the backend directory
cd backend

# Build the project (skip tests for faster startup)
mvn clean install -DskipTests

# Run the application
mvn spring-boot:run
```

The backend starts at: **`http://localhost:8080/api`**

Hibernate auto-creates all tables. A `DataInitializer` seeds the following default users on startup:

| Username | Password | Role |
|---|---|---|
| `admin` | `admin123` | `ADMIN` |
| `officer1` | `officer123` | `ADMISSION_OFFICER` |
| `management` | `mgmt123` | `MANAGEMENT` |

---

### Step 4 — Run the Frontend

No build step required — the frontend is pure HTML/CSS/JS.

**Option 1: VS Code Live Server (Recommended)**
```
Right-click frontend/index.html → "Open with Live Server"
```

**Option 2: Python HTTP server**
```bash
cd frontend
python3 -m http.server 5500
# Open: http://localhost:5500
```

**Option 3: Node http-server**
```bash
npx http-server frontend -p 5500
# Open: http://localhost:5500
```

---

## 🗂️ Project Structure

```
admission-crm/
├── backend/
│   ├── pom.xml
│   └── src/main/java/com/edumerge/admission/
│       ├── AdmissionCrmApplication.java        ← Entry point
│       ├── config/
│       │   ├── SecurityConfig.java             ← JWT + BCrypt + CORS setup
│       │   └── DataInitializer.java            ← Default user seeding
│       ├── controller/
│       │   ├── AuthController.java             ← Login, Register
│       │   ├── MasterController.java           ← Institution, Campus, Dept, Program
│       │   └── AdmissionController.java        ← Applicants, Seats, Confirm
│       ├── service/
│       │   ├── AuthService.java                ← Login + BCrypt logic
│       │   ├── MasterService.java              ← Master data CRUD
│       │   └── AdmissionService.java           ← Core admission workflow
│       ├── entity/
│       │   ├── User.java
│       │   ├── Institution.java, Campus.java
│       │   ├── Department.java, Program.java
│       │   ├── Applicant.java, Admission.java
│       ├── repository/                         ← JPA interfaces (auto-query generation)
│       ├── security/
│       │   ├── JwtUtils.java                   ← Token generation & validation
│       │   ├── JwtAuthFilter.java              ← Per-request token check
│       │   └── CustomUserDetailsService.java
│       ├── dto/                                ← Request/Response data objects
│       └── exception/                          ← Custom exceptions + global handler
│
└── frontend/
    ├── index.html                              ← Login page
    ├── pages/dashboard.html                   ← Main SPA shell
    ├── css/style.css                          ← All styles
    └── js/
        ├── api.js                             ← HTTP calls + JWT injection
        ├── auth.js                            ← Login / logout / session
        └── app.js                             ← All UI logic
```

---

## 🔐 API Endpoints

### Authentication (Public)

```
POST   /api/auth/login       → Returns JWT token
POST   /api/auth/register    → Create user (Admin only)
```

### Master Data Setup (Admin Only)

```
POST/GET   /api/institutions
POST/GET   /api/campuses?institutionId=1
POST/GET   /api/departments?campusId=1
POST/GET   /api/programs
```

### Admission Workflow (Officer / Admin)

```
POST   /api/applicants                         → Create applicant
GET    /api/applicants                         → List all applicants
GET    /api/applicants/{id}                    → Get single applicant
POST   /api/applicants/{id}/allocate-seat      → Lock seat in quota
PUT    /api/applicants/{id}/document-status    → Update document verification
PUT    /api/applicants/{id}/fee-status         → Mark fee as paid
POST   /api/applicants/{id}/confirm            → Generate admission number
GET    /api/dashboard                          → Stats & seat matrix
```

---

## 🧪 Testing the API with curl

```bash
# 1. Login and get a JWT token
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'

# 2. Use the returned token in subsequent requests
curl http://localhost:8080/api/dashboard \
  -H "Authorization: Bearer YOUR_JWT_TOKEN_HERE"

# 3. Create an institution
curl -X POST http://localhost:8080/api/institutions \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"RV College of Engineering","code":"RVCE","email":"info@rvce.edu.in"}'
```

---

## 🔑 JWT Authentication Flow

```
1.  POST /auth/login { username, password }
2.  Server validates via BCrypt → returns signed JWT
3.  Client stores JWT in localStorage
4.  Every request: Authorization: Bearer <token>
5.  JwtAuthFilter validates token → sets SecurityContext
6.  @PreAuthorize checks role from SecurityContext
```

**Token structure:** `HEADER.PAYLOAD.SIGNATURE`
The signature is HMAC-SHA256 signed with a server-only secret — tamper-proof.

---

## 📋 Business Rules

| # | Rule |
|---|---|
| 1 | `kcetSeats + comedkSeats + managementSeats` must equal `totalIntake` (enforced on program creation) |
| 2 | Seat allocation is **blocked** if the quota is full → HTTP 409 (`QuotaFullException`) |
| 3 | Admission number is generated **only once** — immutable (`@Column(updatable = false)`) |
| 4 | Admission can only be **confirmed after fee = PAID** |
| 5 | Seat counters are updated atomically with `@Transactional` — no partial state |

---

## 🔄 Applicant Lifecycle

```
APPLIED
  ↓  (allocate-seat endpoint)
SEAT_ALLOCATED  ← quota counter incremented
  ↓  (document-status → VERIFIED)
DOCUMENTS_VERIFIED
  ↓  (fee-status → PAID)
  ↓  (confirm endpoint)
ADMITTED  ← admission number generated, immutable
```

---

## 🎓 Admission Number Format

```
RVCE / 2026 / UG / CSE / KCET / 0001
 ↑      ↑      ↑    ↑     ↑       ↑
 Inst  Year  Type  Prog  Quota  Sequence
```

- **Unique** — enforced by `@Column(unique = true)` at the DB level
- **Immutable** — `@Column(updatable = false)` prevents any Hibernate UPDATE
- **Self-documenting** — encodes full context in the string

---

## 🏗️ Architecture — SOLID Principles

| Principle | Where Applied | How |
|---|---|---|
| **S** — Single Responsibility | `JwtUtils`, `AuthService` | Each class has exactly one job |
| **O** — Open/Closed | Seat allocation `switch` on `QuotaType` | New quota type = new case only, no rewrite |
| **L** — Liskov Substitution | `CustomUserDetailsService` | Implements `UserDetailsService` — Spring Security uses it transparently |
| **I** — Interface Segregation | All Repository interfaces | Each declares only the methods it actually needs |
| **D** — Dependency Inversion | Services → Repository interfaces | Spring injects the concrete implementation at runtime |

---

## 🗄️ Database Schema

```
institutions (id, name, code, email, active)
  └─ campuses (id, name, location, institution_id)
       └─ departments (id, name, code, campus_id)
            └─ programs (id, name, code, course_type, total_intake,
                         kcet_seats, kcet_allocated,
                         comedk_seats, comedk_allocated,
                         management_seats, management_allocated)
                 └─ applicants (id, first_name, last_name, email, phone,
                                category, quota_type, status,
                                document_status, fee_status, program_id)
                       └─ admissions (id, admission_number [UNIQUE, immutable],
                                      applicant_id, program_id, confirmed_at)

users (id, username, password [BCrypt hash], role, active)
```

---

## ⚠️ Common Issues & Fixes

**Backend fails to start — DB connection refused**
- Make sure MySQL is running
- Double-check `spring.datasource.username` and `spring.datasource.password` in `application.properties`

**Tables not created automatically**
- Confirm `spring.jpa.hibernate.ddl-auto=update` is set
- The database `admission_crm` must exist before starting (Hibernate does not create the database, only the tables)

**Frontend shows 401 on all requests**
- JWT token may be expired (expires after 24 hours by default)
- Clear `localStorage` and log in again

**Port conflict on 8080**
- Change `server.port` in `application.properties`
- Update the `API_BASE_URL` in `frontend/js/api.js` to match

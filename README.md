# Visitor Registration System

A web-based Visitor Registration System that replaces manual visitor logbooks with digital check-in, QR code generation, host approval workflows, and an admin dashboard.

## Tech Stack

- **Backend**: Node.js + Express.js
- **Database**: SQLite (via sql.js — pure JS, zero native dependencies)
- **Auth**: JWT (JSON Web Tokens) + bcrypt
- **QR Codes**: Generated server-side using `qrcode`
- **Frontend**: Vanilla HTML/CSS/JS (single-page app)
- **Testing**: Playwright (API + E2E, headed mode)
- **Containerization**: Docker

## Project Structure

```
visitor-registration-system/
├── docs/                        # SDLC documentation
│   ├── 01-project-plan.md       # Planning phase
│   ├── 02-requirements-srs.md   # Requirements & SRS
│   ├── 03-system-design.md      # HLD, LLD, DB schema, API design
│   ├── 04-test-plan.md          # STLC test plan
│   ├── 05-deployment-guide.md   # Deployment instructions
│   └── 06-maintenance.md        # Maintenance & roadmap
├── public/                      # Frontend (served as static files)
│   ├── index.html
│   ├── css/style.css
│   └── js/app.js
├── src/                         # Backend source
│   ├── server.js                # Express app entry point
│   ├── database.js              # SQLite DB layer
│   ├── seed.js                  # Seed default users
│   ├── middleware/auth.js       # JWT auth & role middleware
│   └── routes/
│       ├── auth.js              # Login API
│       ├── visitors.js          # Visitor CRUD + workflow APIs
│       └── dashboard.js         # Admin stats API
├── tests/                       # Playwright test suite
│   ├── playwright.config.js
│   ├── api.spec.js              # 13 API tests (unit/integration/security/perf)
│   └── e2e.spec.js              # 5 E2E browser tests
├── data/                        # SQLite DB file (auto-created)
├── Dockerfile
├── .dockerignore
└── package.json
```

## Getting Started

### Prerequisites

- Node.js 18+ installed
- npm

### Installation

```bash
cd visitor-registration-system
npm install
```

### Run the Application

```bash
npm start
```

The server starts at **http://localhost:3000**.

On first launch, it automatically seeds two default users:

| Role     | Username   | Password      |
|----------|------------|---------------|
| Admin    | admin      | admin123      |
| Security | security   | security123   |

You can also seed manually:

```bash
npm run seed
```

### Run in Development Mode (auto-restart on changes)

```bash
npm run dev
```

## Application Workflows

### 1. Visitor Registration (Public)

```
Visitor opens http://localhost:3000
  → Fills registration form (name, email, phone, purpose, host)
  → Submits
  → System generates unique Visit ID + QR code
  → Visitor receives QR code on screen
  → Visit status = "pending"
```

### 2. Host Approval / Rejection

```
Admin logs in → Dashboard
  → Sees pending visitors in the table
  → Clicks "Approve" or "Reject"
  → Visit status updates to "approved" or "rejected"
```

Approval can also be done via API:
```bash
# Approve
curl -X PUT http://localhost:3000/api/visitors/{visit_id}/approve

# Reject
curl -X PUT http://localhost:3000/api/visitors/{visit_id}/reject
```

### 3. Security Check-in / Check-out

```
Security guard navigates to "Check-in" page
  → Enters Visit ID (from QR code scan)
  → Clicks "Lookup" to see visitor details
  → If status is "approved" → "Check In" button appears → Click to check in
  → If status is "checked_in" → "Check Out" button appears → Click to check out
```

Requires security/admin JWT token for check-in/check-out API calls.

### 4. Admin Dashboard

```
Admin logs in → Dashboard page
  → Views stats: Today's visitors, Pending, Checked In, Total
  → Filters by date, status, or search text
  → Approves/rejects pending visitors
  → Exports all visitor data as CSV
```

### 5. Complete Visitor Lifecycle

```
┌──────────┐    ┌──────────┐    ┌────────────┐    ┌─────────────┐
│ Register │───▶│ Pending  │───▶│  Approved  │───▶│ Checked In  │
└──────────┘    └────┬─────┘    └────────────┘    └──────┬──────┘
                     │                                    │
                     ▼                                    ▼
                ┌──────────┐                      ┌─────────────┐
                │ Rejected │                      │ Checked Out │
                └──────────┘                      └─────────────┘
```

## API Reference

| Method | Endpoint                          | Auth       | Description                    |
|--------|-----------------------------------|------------|--------------------------------|
| POST   | `/api/auth/login`                 | No         | Login, returns JWT token       |
| POST   | `/api/visitors`                   | No         | Register a new visitor         |
| GET    | `/api/visitors`                   | Admin/Sec  | List visitors (filterable)     |
| GET    | `/api/visitors/:visitId`          | No         | Get single visitor details     |
| PUT    | `/api/visitors/:visitId/approve`  | No         | Approve a pending visitor      |
| PUT    | `/api/visitors/:visitId/reject`   | No         | Reject a pending visitor       |
| PUT    | `/api/visitors/:visitId/checkin`  | Security   | Check in an approved visitor   |
| PUT    | `/api/visitors/:visitId/checkout` | Security   | Check out a checked-in visitor |
| GET    | `/api/visitors/export/csv`        | Admin      | Export all visitors as CSV     |
| GET    | `/api/dashboard/stats`            | Admin      | Dashboard statistics           |
| GET    | `/api/health`                     | No         | Health check                   |

## Running Tests

Tests use Playwright in headed mode (browser visible). The test suite auto-starts a server on port 3001 with a separate test database.

```bash
npm test
```

This runs 18 tests across two projects:

**API Tests (13 tests)**
- TC-01: Register a new visitor successfully
- TC-02: Prevent duplicate registration same day
- TC-03: Registration fails with missing fields
- TC-04: Admin login returns token
- TC-05: Login fails with wrong password
- TC-06: Approve a pending visitor
- TC-07: Reject a pending visitor
- TC-08: Check-in an approved visitor
- TC-09: Check-out a checked-in visitor
- TC-10: Check-in fails for pending visitor
- TC-14: Dashboard stats require auth
- TC-15: Invalid token is rejected
- TC-16: Handle 20 concurrent registrations (performance)

**E2E Tests (5 tests)**
- TC-11: Register a visitor via UI
- TC-12: Admin login and dashboard loads
- TC-13: Register visitor then approve from dashboard
- TC-E2E-04: Check-in page is accessible
- TC-E2E-05: All public pages are navigable

### View Test Report

After running tests, an HTML report with screenshots, videos, and traces is generated:

```bash
npm run test:report
```

Opens the Playwright HTML report in your browser.

## Docker Deployment

```bash
# Build
docker build -t visitor-registration-system .

# Run
docker run -p 3000:3000 -v vrs-data:/app/data visitor-registration-system
```

## Environment Variables

| Variable     | Default                          | Description          |
|--------------|----------------------------------|----------------------|
| `PORT`       | `3000`                           | Server port          |
| `JWT_SECRET` | `vrs-secret-key-change-in-production` | JWT signing key |
| `DB_PATH`    | `./data/visitors.db`             | SQLite database path |

## SDLC Documentation

Full lifecycle documentation is in the `docs/` folder:

1. **Planning** — Project scope, feasibility analysis, risk register
2. **Requirements** — SRS, functional/non-functional requirements, user stories, use cases
3. **System Design** — Architecture (HLD), database schema & API design (LLD)
4. **Test Plan** — STLC strategy, test cases, entry/exit criteria
5. **Deployment Guide** — Local, Docker, and cloud deployment instructions
6. **Maintenance** — Bug fix process, monitoring, future roadmap

## License

MIT

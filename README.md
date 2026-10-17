# EventPulse — Next-Gen Event Registration & Capacity Platform

[![Node.js](https://img.shields.io/badge/Node.js-v18%2B-green.svg)](https://nodejs.org/)
[![Express.js](https://img.shields.io/badge/Express.js-v4.21-lightgrey.svg)](https://expressjs.com/)
[![React](https://img.shields.io/badge/React-v19.0-blue.svg)](https://react.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4.0-38bdf8.svg)](https://tailwindcss.com/)
[![SQLite3](https://img.shields.io/badge/SQLite-v5.1-003B57.svg)](https://sqlite.org/)
[![Status](https://img.shields.io/badge/Status-Production_Ready-emerald.svg)]()

> A 100% complete, fully functional, production-ready web application for **Task 3: Event Registration Platform**. Engineered with strict duplicate registration prevention (`HTTP 409 Conflict`), atomic dynamic seat decrements, verifiable digital QR check-in passes, ticket cancellation with seat release (`+1`), and an administrative registry dashboard with real-time CSV export.

---

## Table of Contents
1. [Architecture & System Design](#architecture--system-design)
2. [Monorepo Directory Structure](#monorepo-directory-structure)
3. [Database Schema & ER Diagram](#database-schema--er-diagram)
4. [Core Features & Engineering Highlights](#core-features--engineering-highlights)
5. [Prerequisites & Quick Start](#prerequisites--quick-start)
6. [API Route Reference](#api-route-reference)
7. [Automated Verification & Integration Tests](#automated-verification--integration-tests)
8. [License & Authors](#license--authors)

---

## Architecture & System Design

```
                     ┌──────────────────────────────────────────┐
                     │          React 19 + Tailwind v4          │
                     │          Vite SPA Client (Port 3000)     │
                     └────────────────────┬─────────────────────┘
                                          │
                                HTTP REST │ (Vite Proxy: /api -> :5000)
                                          ▼
                     ┌──────────────────────────────────────────┐
                     │           Node.js + Express.js           │
                     │           REST API Server (Port 5000)    │
                     └────────────────────┬─────────────────────┘
                                          │
                        Transactions & PRAGMA foreign_keys = ON
                                          ▼
                     ┌──────────────────────────────────────────┐
                     │         Persistent SQLite Database       │
                     │          (server/db/event_platform.db)   │
                     └──────────────────────────────────────────┘
```

- **Frontend**: React 19 SPA running on Vite 6 with Tailwind CSS v4, Lucide React icons, and Canvas Confetti. Features a sleek modern dark-mode aesthetic with custom glassmorphism and real-time client state updates.
- **Backend**: Node.js Express server structured into controllers, routes, database drivers, and validation middleware.
- **Database**: SQLite3 with persistent on-disk storage, transactional concurrency controls (`BEGIN TRANSACTION` / `COMMIT`), unique email-per-event constraints, and database cascading rules.

---

## Monorepo Directory Structure

```
Event Registration Platform/
├── package.json                    # Root workspace package orchestrating dev servers
├── README.md                       # Comprehensive system architecture & setup documentation
├── client/                         # Frontend Application (React 19 + Tailwind CSS)
│   ├── index.html                  # HTML5 entry with Google Fonts (Plus Jakarta Sans)
│   ├── package.json                # Client dependencies & Vite build scripts
│   ├── vite.config.js              # Vite configuration with Tailwind v4 & proxy setup
│   └── src/
│       ├── main.jsx                # Application root mounting React 19 DOM
│       ├── App.jsx                 # Master application controller & state orchestrator
│       ├── index.css               # Core styling tokens, glassmorphism, & scrollbars
│       ├── components/
│       │   ├── Navbar.jsx          # Sticky header with navigation & ticket quick-lookup
│       │   ├── Footer.jsx          # Technical footer with stack badges & constraints
│       │   ├── EventCard.jsx       # Public showcase card with occupancy progress bar
│       │   ├── RegistrationModal.jsx # Attendee form with domain pills & 409 error banner
│       │   ├── RegistrationTicket.jsx# Printable digital ticket pass with dynamic QR code
│       │   ├── TicketLookupModal.jsx # Ticket code / email lookup & cancellation dialog
│       │   ├── EventDetailsModal.jsx # Detailed event agenda, speaker, & topics modal
│       │   ├── CreateEventModal.jsx  # Organizer dialog for creating new events
│       │   └── AdminDashboard.jsx    # Tabular attendee inspection, check-in, & CSV export
│       └── services/
│           └── api.js              # Centralized client API service with error mapping
└── server/                         # Backend Application (Node.js + Express + SQLite)
    ├── package.json                # Backend dependencies & run scripts
    ├── server.js                   # Express application bootstrap & route mounts
    ├── test_api.js                 # 12-step end-to-end integration test runner
    ├── .env                        # Local configuration variables
    ├── controllers/
    │   ├── eventController.js      # Event retrieval, filtering, creation, & categories
    │   ├── registrationController.js# Registration, 409 duplicate check, cancel & tickets
    │   └── adminController.js      # Attendee registry, status update, stats, & CSV export
    ├── routes/
    │   ├── eventRoutes.js          # /api/events routes
    │   ├── registrationRoutes.js   # /api/registrations routes
    │   └── adminRoutes.js          # /api/admin routes
    ├── middleware/
    │   ├── validator.js            # Input validation & schema sanitization
    │   └── errorHandler.js         # Centralized error handler
    └── db/
        ├── database.js             # SQLite connection pool & async Promise wrapper
        ├── schema.sql              # DDL schema definitions, constraints, & indexes
        ├── seed.js                 # Database seeder with 6 rich events & 24 attendees
        └── event_platform.db       # Persistent SQLite database file
```

---

## Database Schema & ER Diagram

The database utilizes SQLite with `PRAGMA foreign_keys = ON` and explicit unique constraints to enforce data integrity at the storage layer.

```
┌─────────────────────────────────┐               ┌─────────────────────────────────┐
│             events              │               │          registrations          │
├─────────────────────────────────┤               ├─────────────────────────────────┤
│ id (PK)             INTEGER     │ 1           * │ id (PK)             INTEGER     │
│ title               TEXT        │◄─────────────┤ ticket_code (UNIQUE)TEXT       │
│ slug (UNIQUE)       TEXT        │               │ event_id (FK)       INTEGER     │
│ description         TEXT        │               │ full_name           TEXT        │
│ category            TEXT        │               │ email               TEXT        │
│ date                TEXT        │               │ phone               TEXT        │
│ time                TEXT        │               │ college_organizationTEXT        │
│ location            TEXT        │               │ domain_interests    TEXT        │
│ is_virtual          INTEGER     │               │ notes               TEXT        │
│ capacity            INTEGER     │               │ status              TEXT        │
│ registered_count    INTEGER     │               │ registered_at       DATETIME    │
│ speaker_name        TEXT        │               ├─────────────────────────────────┤
│ speaker_role        TEXT        │               │ UNIQUE(event_id, email)         │
│ image_url           TEXT        │               └─────────────────────────────────┘
│ tags                TEXT        │
│ created_at          DATETIME    │
└─────────────────────────────────┘
```

### Table Definitions (`server/db/schema.sql`):
1. **`events`**:
   - `id`: Primary Key, auto-incrementing integer.
   - `title`, `slug`, `description`, `category`, `date`, `time`, `location`.
   - `capacity`: Maximum allowed attendees (`CHECK (capacity > 0)`).
   - `registered_count`: Active confirmed seats (`CHECK (registered_count >= 0)`).
   - `is_virtual`: 0 for in-person, 1 for virtual livestream.
   - `speaker_name`, `speaker_role`, `image_url`, `tags` (JSON string array).
2. **`registrations`**:
   - `id`: Primary Key, auto-incrementing integer.
   - `ticket_code`: Unique formatted identifier (e.g. `REG-AI-D4843C`).
   - `event_id`: Foreign key referencing `events(id)` with `ON DELETE CASCADE`.
   - `full_name`, `email`, `phone`, `college_organization`.
   - `domain_interests`: Serialized JSON array of chosen specializations.
   - `status`: Enum constraint (`CHECK(status IN ('Confirmed', 'Checked In', 'Cancelled'))`).
   - `CONSTRAINT unique_event_attendee_email UNIQUE (event_id, email)`: **Prevents duplicate registrations at the database level**.

---

## Core Features & Engineering Highlights

### 1. Public Event Showcase & Dynamic Filtering
- Real-time search across event titles, descriptions, speakers, and venues.
- Category filters: *Artificial Intelligence*, *Cloud & DevOps*, *Web Development*, *Cybersecurity*, *UI/UX Design*, *Emerging Tech*.
- Date filters: *All Dates*, *Upcoming Only*, *This Month*.
- Dynamic seat counter progress bar transitioning from indigo/cyan to amber (when spots are low) and red (when sold out).

### 2. Strict Duplicate Check (`HTTP 409 Conflict`)
- Every registration attempt undergoes a pre-flight query checking if `LOWER(email)` is already registered for that specific `event_id`.
- If found, the server immediately rejects the request with an informative `409 Conflict` status code:
  ```json
  {
    "success": false,
    "error": "DUPLICATE_REGISTRATION",
    "message": "The email address 'alex@university.edu' is already registered for \"Global AI Summit 2026\". Duplicate registrations are not permitted.",
    "details": {
      "ticketCode": "REG-AI-8F21B9",
      "registeredAt": "2026-10-17 10:00:00"
    }
  }
  ```
- The frontend renders an amber/red alert banner and offers a one-click button to view the existing ticket pass.

### 3. Dynamic Capacity Management
- **Upon Registration**: An atomic SQLite transaction inserts the registration and increments the event's `registered_count` by 1 (`UPDATE events SET registered_count = registered_count + 1 WHERE id = ?`).
- **Capacity Lock**: If `registered_count >= capacity`, the registration is blocked with a `400 Bad Request` and `CAPACITY_REACHED` error.
- **Client Sync**: Returns `event_updates` so the UI decrements the remaining capacity immediately in real time.

### 4. Success Modal & Dynamic QR Pass
- Successful registration displays a confetti celebration and issues a printable ticket pass.
- Features a dynamic QR code rendered via SVG/image encoding `https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=<TICKET_CODE>`.
- Includes attendee credentials, ticket ID with copy button, event date/time, venue, and a "Print / Save Pass" action.

### 5. Ticket Cancellation / Unregister Flow
- Attendees can look up their ticket by **Ticket Code** or **Registered Email** and cancel their registration.
- Safely deletes the attendee registration record and increments the event's available capacity back (`+1`) atomically.
- Frees the email so the attendee can register again in the future if desired.

### 6. Administrative Dashboard & Excel/CSV Export
- Tabular inspection view listing all registered attendees with search and status badges (*Confirmed*, *Checked In*, *Cancelled*).
- Single-click attendee check-in toggle and status management.
- Direct **"Export Attendees to CSV"** button generating an RFC-4180 compliant CSV file with properly escaped headers and values.
- Dashboard analytics: Total registrations, overall capacity fill rate %, unique attendees, and institutional diversity metrics.

---

## Prerequisites & Quick Start

### Prerequisites
- [Node.js](https://nodejs.org/) v18.0.0 or higher
- `npm` v9.0.0 or higher

### 1. Installation
In the project root directory, install all dependencies for both client and server:
```bash
# Install root, backend, and frontend dependencies
npm run install:all
```
*Or install individually:*
```bash
# Server dependencies
cd server && npm install

# Client dependencies
cd ../client && npm install
```

### 2. Seed Database
Seed the SQLite database with 6 curated events and 24 attendee records:
```bash
npm run seed
```
*(Direct script: `node server/db/seed.js`)*

### 3. Start Development Servers
Start both the Express API and Vite React frontend concurrently with one command:
```bash
npm run dev
```

Alternatively, run each service in separate terminal windows:
```bash
# Terminal 1: Start Backend (Port 5000)
npm run server
# (or: cd server && npm run dev)

# Terminal 2: Start Frontend (Port 3000)
npm run client
# (or: cd client && npm run dev)
```

- **Frontend Application**: [http://localhost:3000](http://localhost:3000)
- **Backend API**: [http://localhost:5000](http://localhost:5000)
- **Health Check**: [http://localhost:5000/api/health](http://localhost:5000/api/health)

---

## API Route Reference

### Events (`/api/events`)
| Method | Endpoint | Description | Query / Body Params |
|---|---|---|---|
| `GET` | `/api/events` | List all events with live capacities | `?category=AI&search=summit&status=open` |
| `GET` | `/api/events/:id` | Get single event details with attendee stats | Path: `:id` |
| `GET` | `/api/events/categories` | Get category breakdown & counts | None |
| `POST` | `/api/events` | Create a new event | Body: `title, category, date, time, location, capacity, ...` |

### Registrations (`/api/registrations`)
| Method | Endpoint | Description | Response Status |
|---|---|---|---|
| `POST` | `/api/registrations` | Register attendee for an event (decrements capacity -1) | `201 Created` or `409 Conflict` |
| `GET` | `/api/registrations/ticket/:ticketCode` | Lookup ticket details by unique code | `200 OK` or `404 Not Found` |
| `POST` | `/api/registrations/cancel` | Cancel registration & restore capacity (+1) | `200 OK` |
| `DELETE` | `/api/registrations/cancel/:ticketCode`| Cancel registration by ticket code parameter | `200 OK` |
| `GET` | `/api/registrations/attendee/:email` | Find all registrations by attendee email | `200 OK` |

#### Registration Request Payload:
```json
{
  "event_id": 1,
  "full_name": "Elena Rostova",
  "email": "elena.rostova@neuralmesh.ai",
  "phone": "+1 (555) 234-5678",
  "college_organization": "NeuralMesh Research",
  "domain_interests": ["Artificial Intelligence", "Quantum Computing"],
  "notes": "Interested in agentic reasoning keynote"
}
```

### Admin Operations (`/api/admin`)
| Method | Endpoint | Description | Query / Body Params |
|---|---|---|---|
| `GET` | `/api/admin/registrations` | Tabular registry of all attendees | `?search=john&eventId=1&status=Confirmed` |
| `PATCH`| `/api/admin/registrations/:id/status`| Update attendee status (*Confirmed*, *Checked In*, *Cancelled*) | Body: `{ "status": "Checked In" }` |
| `DELETE`| `/api/admin/registrations/:id` | Delete attendee registration & release seat | Path: `:id` |
| `GET` | `/api/admin/stats` | Dashboard metrics & occupancy analytics | None |
| `GET` | `/api/admin/export` | Download full attendee list as CSV | Header: `attachment; filename=event_attendees_export.csv` |

---

## Automated Verification & Integration Tests

The project includes an automated end-to-end integration test runner validating all specifications:
```bash
npm test
```
*(Or directly: `node server/test_api.js`)*

### Tested Scenarios:
1. **Health Check**: Validates API availability on `/api/health`.
2. **Event Retrieval**: Verifies categories and active capacity calculation.
3. **Participant Registration**: Validates successful registration and real-time seat decrement (-1).
4. **Duplicate Prevention**: Re-attempts registration with identical email and asserts `409 Conflict`.
5. **Sold-Out Enforcement**: Attempts registration on a full event (Event 4) and asserts `400 Bad Request`.
6. **Digital Pass Ticket Lookup**: Verifies ticket code retrieval.
7. **Admin Dashboard Stats**: Confirms total counts, fill rate percentages, and institution counts.
8. **Admin Attendee Inspection**: Inspects attendee registry table.
9. **Attendee Status Update**: Checks in an attendee.
10. **CSV Export**: Validates `text/csv` headers and RFC-4180 row formatting.
11. **Ticket Cancellation Flow**: Unregisters an attendee, deletes record, and confirms available capacity increments back (+1).
12. **Post-Cancellation Re-registration**: Verifies the email is released and can register again cleanly.

---

## Production Build

To produce an optimized production bundle of the client:
```bash
npm run build
```
The compiled assets will be written to `client/dist/`.

---

## Authors & Acknowledgments
Built with modern software engineering best practices for the **Event Registration Platform (Task 3)** specification.

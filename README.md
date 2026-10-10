# CampusSafe — Smart Campus Emergency Reporting & Response Management

A polished, animated, production-style MERN platform for campus emergencies: report-to-resolution workflow with real-time alerts, live maps and analytics.

## Architecture

```
Smart Campus TT/
├── frontend/          # React + Vite + Tailwind + Framer Motion + Leaflet + Recharts
│   └── src/
│       ├── components/  Navbar, Footer, ProtectedRoute, IncidentMap
│       ├── pages/       Landing, Login, Register, Student/Security/Admin dashboards,
│       │                ReportEmergency, TrackIncident, IncidentDetails, MapView, Profile
│       ├── layouts/     PublicLayout, DashboardLayout
│       ├── context/     Auth, Theme, Socket, Toast
│       ├── services/    api (axios + JWT)
│       ├── animations/  variants
│       └── utils/       helpers (colors, CSV, dates)
└── backend/           # Express + Mongoose + JWT + Socket.IO + Multer
    ├── config/ models/ controllers/ routes/ middleware/ sockets/ utils/ scripts/
    └── server.js
```

**Data flow:** Student submits SOS → `POST /api/incidents` → MongoDB + `incident:new` socket event to `role:security/admin` rooms → Security acknowledges/assigns/resolves → `incident:updated` pushed to reporter + roles → Notifications persisted + toasts + bell badge.

## Prerequisites

- Node.js 18+, MongoDB (local or Atlas)

## Run locally

### 1. Backend
```bash
cd backend
cp .env.example .env        # fill MONGO_URI, JWT_SECRET
npm install
npm run seed                # creates demo users + incidents (demo data)
node scripts/createAdmin.js # or ensure admin exists
npm run dev                 # http://localhost:5000
```

Demo logins after seeding: `student@campus.edu`, `faculty@campus.edu`, `security@campus.edu` / `Password@123`; admin = your `ADMIN_SEED_*`.

### 2. Frontend
```bash
cd frontend
cp .env.example .env
npm install
npm run dev                 # http://localhost:5173
```

## Key API routes

| Method | Route | Access |
|---|---|---|
| POST | `/api/auth/register` (student/faculty/security only) | public |
| POST | `/api/auth/login` | public |
| POST | `/api/incidents` (multipart, image ≤5MB) | student/faculty |
| GET | `/api/incidents/my`, `/my/stats` | owner |
| GET | `/api/incidents?active=true&priority=&status=` | security/admin |
| GET/PATCH | `/api/incidents/:id`, `/:id/status` | role-checked |
| GET | `/api/admin/stats`, `/admin/users` | admin |
| GET | `/api/notifications` | auth (role-filtered) |

Socket.IO namespaces: rooms `role:<role>`, `user:<id>`; events `notification`, `incident:new`, `incident:updated`. Frontend polls `/notifications` every 30s as fallback.

## Notes & safety

- Landing-page statistics are **clearly labeled demo data** until the backend provides real counts.
- Emergency disclaimer shown on landing, dashboards and report form: this app **supplements, does not replace** local emergency services.
- Admins cannot self-register; use `scripts/createAdmin.js` or `seed.js`.
- Image uploads stored in `backend/uploads/` (gitignored); serve via `/uploads/<file>`.
- Passwords hashed with bcrypt; JWT in `Authorization: Bearer`; rate limits on auth + report endpoints; backend enforces all role checks.
- Map default center is a demo coordinate — update `CAMPUS_CENTER` in `IncidentMap.jsx` to your campus.

# CampusSafe

CampusSafe is a campus emergency reporting and response application with a React frontend and Spring Boot backend. The existing frontend pages, components, and styling are retained.

## Stack

- Frontend: React 18, Vite, Tailwind CSS, React Router, Leaflet, Recharts
- Backend: Java 17, Spring Boot 3, Spring Data MongoDB
- Database: MongoDB
- Authentication: JWT bearer tokens and BCrypt password hashes

## Run locally

### Backend

Install Java 17+, Maven, and MongoDB. From `backend/`, configure `MONGO_URI` and `JWT_SECRET`, then run:

```bash
mvn spring-boot:run
```

The API starts at `http://localhost:5000`. Defaults are `mongodb://127.0.0.1:27017/campussafe` and port `5000`. Set `CLIENT_URL` if the Vite frontend uses a different origin. Uploaded images are stored under `backend/uploads/` by default.

### Frontend

From `frontend/`, run:

```bash
npm install
npm run dev
```

Vite serves the app at `http://localhost:3000`. Set `VITE_API_URL` to override the API base URL (default `http://localhost:5000/api`).

## API

The Spring Boot API preserves the frontend's `/api` routes for authentication, incident reporting and tracking, notifications, response teams, locations, and admin dashboards. Requests use `Authorization: Bearer <token>`. Notification updates are polled every 10 seconds; the previous Socket.IO transport was removed during the backend migration.

The API routes and core document fields remain compatible with the previous backend. Admin accounts must be created through a trusted provisioning process; public registration accepts student, faculty, and security roles only.

## Deploy the backend to Render

The Render Blueprint at the repository root creates a PostgreSQL database and a Docker web service in Singapore. The service uses `backend/` as its root directory, exposes `/api/health` as its health check, and sets the Vercel frontend origin for CORS. In Render, choose **New + → Blueprint**, connect this repository, and apply the Blueprint. Render will generate `JWT_SECRET` and connect the service to the database. The Blueprint currently selects Render's free plans for a demo: free Postgres expires after 30 days, so select a paid database plan before applying it if you need to keep real data long-term.

After Render deploys the service, copy its `https://…onrender.com` URL into the Vercel project's `VITE_API_URL` environment variable with `/api` at the end (for example, `https://your-api.onrender.com/api`), then redeploy the frontend. The free Render web service may sleep while idle; uploaded files are stored on the service filesystem and are not durable across rebuilds or restarts without persistent storage.

PostgreSQL stores application records in a JSONB document table so the existing API fields remain compatible. The old MongoDB data is not migrated automatically; the Render database starts empty. Create the initial admin account through a trusted provisioning process after deployment.

## Emergency guidance

CampusSafe supplements campus response procedures and does not replace local emergency services. For immediate danger, contact local emergency services first.

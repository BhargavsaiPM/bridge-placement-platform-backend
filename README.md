# Bridge Placement Platform Backend

Backend for the Bridge “Placement” platform. Built with Spring Boot (REST + JWT auth) and backed by PostgreSQL (JPA).

## Tech Stack

- Spring Boot 3.2.3
- Java 21
- Spring Security (JWT, stateless)
- Spring Data JPA (Hibernate)
- PostgreSQL
- Cloudinary (file upload)
- Gemini API (ATS scoring)
- Maven

## Base URL

The server runs with `server.servlet.context-path=/api`, so all endpoints are under:
`http://localhost:<PORT>/api`

Default local port: `9092`.

## Roles / Authorization

Access is role-based (JWT):

- `USER` (student)
- `PLACEMENT_OFFICER` (officer)
- `COMPANY` (company)
- `SUPER_ADMIN` (admin)

Most endpoints require `Authorization: Bearer <JWT>`.

## Prerequisites

- Java 21 installed
- PostgreSQL running with the database `Bridge_PlacementSystem` (local)
- (For production / cloud) required environment variables (see below)

## Quick Start (Local)

1. Start PostgreSQL and create the database:
   - Database: `Bridge_PlacementSystem`
   - Username: `postgres`
   - Password: `root`
2. Run the app:

```powershell
mvn clean spring-boot:run
```

Local profile is selected via `SPRING_PROFILES_ACTIVE` and defaults to `local`.

## Configuration

### Environment Variables

The project uses `.env.example` as a reference. Copy it to `.env` (or set variables in your shell) and fill in the values for your environment.

Production profile is activated with:
- `SPRING_PROFILES_ACTIVE=prod`

Key variables:
- `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`
- `JWT_SECRET` (must be 32+ characters in production)
- `MAIL_USERNAME`, `MAIL_PASSWORD`
- `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` (required for `/files/upload`)
- `GEMINI_API_KEY` (required for ATS score calculations)
- Optional: `APP_DEMO_SEED_ENABLED` (controls demo seed)
- `FRONTEND_URL` (for CORS; also exposed as `bridge.app.frontendUrl` via config)

### CORS

Allowed origins are restricted to the configured frontend URL.

## Docker

```powershell
docker build -t bridge-placement-backend .
docker run -p 9092:9092 -e SPRING_PROFILES_ACTIVE=prod bridge-placement-backend
```

In production, you must also pass required env vars (DB, JWT secret, Cloudinary, Gemini, mail, etc.).

## API Overview (Route Groups)

All endpoints are under `/api`.

### Authentication (Public)
- `POST /auth/login`
- `POST /auth/register-user`
- `POST /auth/register-company`
- `POST /auth/forgot-password`
- `POST /auth/reset-password`

### Public
- `GET /public/stats`
- `GET /jobs/search?location=<...>&type=<...>` (job search)
- `GET /jobs/{id}` (job details)

### Student / User (`USER`)
- `GET /user/profile`
- `PUT /user/profile`
- `GET /user/applications`
- `POST /user/apply/{jobId}`
- `GET /user/application/{applicationId}/interview`
- `GET /user/notifications`
- `PUT /user/notifications/read?notificationId=<id>`
- `GET /user/notifications/unread-count`
- `GET /ats/calculate/{jobId}` (ATS score breakdown using Gemini)

### Placement Officer (`PLACEMENT_OFFICER`)
- `GET /officer/profile`
- `PUT /officer/profile`
- `POST /officer/change-password`
- `GET /officer/jobs`
- `GET /officer/application/{id}`
- `GET /officer/applications/{jobId}` (paginated)
- `PUT /officer/application/status?applicationId=<id>&status=<STATUS>`
- `PUT /officer/application/{id}/remark` (body: `{ "remark": "..." }`)
- `POST /officer/application/{applicationId}/schedule-interview`
- `GET /officer/application/{applicationId}/interview-slots`
- `GET /officer/reports/placement`

### Company (`COMPANY`)
- `POST /company/create-placement-officer`
- `GET /company/profile`
- `PUT /company/profile`
- `GET /company/officers`
- `GET /company/job/{jobId}/applications`
- `GET /company/selected-students`
- `PUT /company/application/{applicationId}/status` (body: `{ "status": "<STATUS>" }`)
- `GET /company/dashboard`
- `PUT /company/officer/{officerId}/deactivate`
- `PUT /company/officer/{officerId}/activate`
- `PUT /company/officer/{officerId}/reset-password` (body: `{ "newPassword": "..." }`)

### Admin (`SUPER_ADMIN`)
- `GET /admin/profile`
- `PUT /admin/profile`
- `GET /admin/stats`
- Company approval/rejection/block:
  - `GET /admin/companies/pending`
  - `POST /admin/company/{id}/approve`
  - `POST /admin/company/{id}/reject`
  - `POST /admin/company/{id}/block`
- User approval/rejection/block:
  - `GET /admin/users/pending?type=<USER_TYPE>`
  - `POST /admin/user/{id}/approve`
  - `POST /admin/user/{id}/reject`
  - `POST /admin/user/{id}/block`
- Officer approval/rejection/block:
  - `GET /admin/officers/pending`
  - `POST /admin/officer/{id}/approve`
  - `POST /admin/officer/{id}/reject`
  - `POST /admin/officer/{id}/block`
- Analytics / dashboards:
  - `GET /admin/placement-stats`
  - `GET /admin/student-performance`
  - `GET /admin/recruiter-engagement`
  - `GET /admin/active-users`
  - `GET /admin/login-logs`
  - `GET /admin/server-load`
- Admin “kanban”:
  - `GET /admin/student-progress?page=<n>&size=<n>`
  - `PUT /admin/student-progress/{id}` (body includes `status`)
- Delete operations:
  - `DELETE /admin/user/{id}`
  - `DELETE /admin/company/{id}`
  - `DELETE /admin/job/{id}`
  - `DELETE /admin/officer/{id}`

### File Upload
- `POST /files/upload` (multipart form field: `file`)

Uploads are handled by `CloudinaryService` and return a JSON response:
`{ "url": "<secure_cloudinary_url>" }`

Multipart upload limits:
- `spring.servlet.multipart.max-file-size=5MB`
- `spring.servlet.multipart.max-request-size=5MB`

## Demo Data Seed (Optional)

If enabled (`app.demo.seed.enabled=true`), the backend will pre-populate the database with demo users/companies/jobs.

## Notes / Common Issues

- Remember the `/api` prefix in all calls.
- In production, ensure `JWT_SECRET` is 32+ characters.
- Ensure Cloudinary env vars are set for `/files/upload`.
- Ensure `GEMINI_API_KEY` is set for ATS scoring.


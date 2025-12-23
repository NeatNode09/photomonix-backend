# Photomonix Auth API

Node.js + Express authentication service using Drizzle ORM (Postgres/Neon), JWT access/refresh tokens, and email workflows for verification and password reset.

## Stack

- TypeScript, Express 5, Drizzle ORM (PostgreSQL via Neon HTTP driver)
- JWT (access + refresh), bcrypt
- Nodemailer for verification and reset emails

## Prerequisites

- Node.js 22+
- PostgreSQL database URL (Neon or compatible)

## Environment

Create a `.env` at the repo root. Required keys:

- `PORT` (default 5000)
- `DATABASE_URL` (Postgres connection string)
- `NODE_ENV` (development|production)
- `CORS_ORIGIN` or `CORS_ORIGINS` (comma-separated list; defaults include http://localhost:3000 and http://localhost:8000)
- `UPLOAD_LIMIT` (MB, default 10)
- `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET` (or legacy `JWT_SECRET`)
- `JWT_ACCESS_EXPIRES_IN`, `JWT_REFRESH_EXPIRES_IN` (e.g., 1d, 7d)
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`, `EMAIL_FROM`
- `FRONTEND_URL` (used to build verification/reset links)

## Setup

```bash
npm install
```

### Docker / Compose

- Build and run (detached): `docker compose up -d --build`
- View logs: `docker compose logs -f api`
- Run migrations (if desired) inside the container once the DB is reachable: `docker compose run --rm api npm run db:migrate`.
- Stop containers: `docker compose down` (keeps the `db_data` volume)
- API container uses `restart: unless-stopped`; keep it running for background tasks.

### Database (Drizzle)

- Update `DATABASE_URL` in `.env`.
- Generate SQL from schemas: `npm run db:generate`
- Apply migrations: `npm run db:migrate`
- Push schema directly (optional): `npm run db:push`
- Inspect data: `npm run db:studio`

### Run

- Dev: `npm run dev` (nodemon + ts-node)
- Build: `npm run build`
- Start (after build): `npm start`

## API

- `GET /health` – service status
- `POST /api/auth/register` – create account; sends verification email
- `POST /api/auth/login` – returns access token (JSON) and refresh token (HttpOnly cookie on `/api/auth`)
- `POST /api/auth/verify-email` – verify via token
- `POST /api/auth/resend-verification`
- `POST /api/auth/refresh` – rotate refresh + issue new access token
- `POST /api/auth/logout` – clears refresh cookie
- Password reset: `POST /api/auth/forgot-password`, `POST /api/auth/reset-password-with-token`
- Profile: `GET /api/auth/profile`, `PUT /api/auth/update-profile` (requires access token)

## Notes

- Email verification guard in middleware is currently commented out; decide policy before production.
- Refresh tokens are stored hashed; cookies are scoped to `/api/auth` and `httpOnly` with `sameSite=strict` (secure in production).
- Drizzle schema lives in `src/models` and outputs migrations to `drizzle/`.

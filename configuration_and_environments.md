# Configuration & Environments
## Fleet Management System

| Field | Value |
|---|---|
| Version | 1.0 (Draft) |
| Date | 2026-10-10 |
| Status | Draft — ready for review |
| Related | docs/architecture_specification.md, docs/api_integration_guide.md, docs/PROJECT-LOG.md |

> Covers: environment variables for every app, how the UI stores the API base URL,
> and the backend config approach. **No secrets are ever committed to git.**

---

## 1. Principles
- All environment-specific values come from env vars / env files, never hard-coded.
- Secrets (JWT secret, DB URL, R2 keys) live only in the deployment platform's env
  settings and in local `.env` files that are git-ignored.
- Commit a `.env.example` (no real values) so everyone knows what is required.

---

## 2. Backend — environment variables (FastAPI)

Loaded via `pydantic-settings` from environment / `.env` (local only).

| Variable | Example (placeholder) | Purpose |
|---|---|---|
| `DATABASE_URL` | `postgresql+asyncpg://user:pass@host/db` | Neon PostgreSQL connection (async). |
| `JWT_SECRET` | `change-me` | Signs access/refresh tokens. |
| `ACCESS_TOKEN_TTL_MIN` | `15` | Access token lifetime (minutes). |
| `REFRESH_TOKEN_TTL_DAYS` | `7` | Refresh token lifetime (days). |
| `R2_ENDPOINT` | `https://<acct>.r2.cloudflarestorage.com` | Cloudflare R2 endpoint. |
| `R2_ACCESS_KEY_ID` | `...` | R2 credentials. |
| `R2_SECRET_ACCESS_KEY` | `...` | R2 credentials. |
| `R2_BUCKET` | `fms-files` | Bucket for photos/docs. |
| `CORS_ORIGINS` | `https://app.example.com` | Allowed web origins (comma-separated). |
| `ENV` | `local` / `production` | Environment name. |

**`backend/.env.example`**
```env
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/fms
JWT_SECRET=change-me
ACCESS_TOKEN_TTL_MIN=15
REFRESH_TOKEN_TTL_DAYS=7
R2_ENDPOINT=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET=fms-files
CORS_ORIGINS=http://localhost:4200
ENV=local
```

> **Best practice (backend):** API endpoint **paths** are defined in code (routers),
> not in env. Only configuration/secrets go in env. This is the recommended approach.

---

## 3. Frontend — API base URL in environment files

Each Angular/Ionic app keeps the **API base URL per environment** in Angular's
`environment.ts` files. The **endpoint paths** live in a typed API client in
`frontend/shared` (not in env) so they're reused and type-checked.

**`src/environments/environment.ts` (dev)**
```ts
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:8000/api/v1',
  staticData: false,        // true = local dummy JSON demo mode (see docs/demo_static_data_mode.md)
  demoRole: 'super_admin',  // role for the fake session when staticData = true
};
```

**`src/environments/environment.prod.ts` (prod)**
```ts
export const environment = {
  production: true,
  apiBaseUrl: 'https://<render-app>.onrender.com/api/v1',
  staticData: false,
  demoRole: 'super_admin',
};
```

**Endpoint paths (shared, in code — not env):** `frontend/shared/api/endpoints.ts`
```ts
export const ENDPOINTS = {
  auth:   { login: '/auth/login', refresh: '/auth/refresh', logout: '/auth/logout', me: '/auth/me' },
  users:  { base: '/users', byId: (id: number) => `/users/${id}` },
  trips:  { base: '/trips', byId: (id: number) => `/trips/${id}`,
            approve: (id: number) => `/trips/${id}/approve` },
  // ...one entry per resource, matching docs/data_model_and_api_contracts.md
};
```

> **Why this split:** base URL changes per environment (dev/prod) → env file.
> Endpoint paths are the same everywhere → code constants (one source, type-safe,
> refactor-friendly). Putting full endpoint strings in env files is **not**
> recommended — it scatters the contract and loses type safety.

### 3.1 Per-app env values
| App | `apiBaseUrl` (dev) | Notes |
|---|---|---|
| Super Admin Web | `http://localhost:8000/api/v1` | Cloudflare Pages in prod. |
| Admin Mobile | same | Capacitor build points to prod URL. |
| User Mobile | same | Capacitor build points to prod URL. |

---

## 4. Git-ignore (must exist)
```
# env / secrets
**/.env
**/.env.*
!**/.env.example
# build output
dist/
node_modules/
__pycache__/
.venv/
```

---

## 5. Where values live per stage
| Value | Local | Production |
|---|---|---|
| Backend secrets | `backend/.env` (git-ignored) | Render env settings |
| DB | local/Neon dev branch | Neon prod |
| Web API base URL | `environment.ts` | `environment.prod.ts` (Cloudflare build) |
| R2 keys | `backend/.env` | Render env settings |

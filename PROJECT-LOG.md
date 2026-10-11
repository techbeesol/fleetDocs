# Project Log – Fleet Management System

This log records approved tools, libraries, and architectural decisions so every agent (Developer, Validator) can rely on them. Keep the Tech Stack section current.

---

## Tech Stack

Legend: **Purpose** · **Version** · **License** · **Reason**

### Backend (Python / FastAPI)

| Tool / Library | Purpose | Version | License | Reason |
|---|---|---|---|---|
| Python | Language runtime | 3.13 | PSF | Approved target runtime. |
| FastAPI | Async REST API framework | latest | MIT | Approved: fast, typed, async. |
| Uvicorn | ASGI server | latest | BSD-3 | Approved; runs FastAPI. |
| Pydantic | Request/response validation (DTOs) | v2 | MIT | Approved; boundary validation. |
| SQLAlchemy (async) | ORM / data access | 2.0 | MIT | **Confirmed by user** as the ORM. |
| asyncpg | Async PostgreSQL driver | latest | Apache-2.0 | Approved; supports Python 3.13. |
| Alembic | Database migrations | latest | MIT | **Confirmed by user** as migration tool; standard SQLAlchemy companion. |
| PyJWT | JWT issue/verify | latest | MIT | Approved; stateless auth. |
| passlib[bcrypt] | Password hashing | latest | BSD | Approved; secure password storage. |
| uv | Dependency/package manager | latest | MIT | Approved tooling. |
| Ruff | Lint/format | latest | MIT | Approved tooling. |
| mypy | Static type checking | latest | MIT | Approved tooling. |
| pre-commit | Git hooks | latest | MIT | Approved tooling. |
| pytest, pytest-cov, pytest-asyncio | Testing + coverage | latest | MIT | Approved; 80% coverage gate. |
| httpx | API E2E test client | latest | BSD-3 | Approved; API-level tests. |

### Frontend – Super Admin Web App (Angular)

| Tool / Library | Purpose | Version | License | Reason |
|---|---|---|---|---|
| Angular | Web framework | 20 | MIT | Approved. |
| Angular Material + CDK | UI components | 20.x | MIT | Approved; **web app only**. |
| AG Grid Community | Data grid | latest | MIT | Free edition only; no enterprise features. |
| Bootstrap | Layout/utilities | 5 | MIT | Approved; styling. |
| SCSS | Styling | — | MIT | Approved. |
| RxJS | Reactive async | latest | Apache-2.0 | Approved. |
| Angular Signals | State management | 20 | MIT | Approved; no NgRx. |

### Frontend – Admin & User Mobile Apps (Ionic)

| Tool / Library | Purpose | Version | License | Reason |
|---|---|---|---|---|
| Angular | App framework | 20 | MIT | Approved. |
| Ionic | UI components | 8 | MIT | Approved; **mobile apps only**. |
| Capacitor | Native shell/packaging | 7 | MIT | Approved; Play Store builds. |
| Angular Signals | State management | 20 | MIT | Approved; no NgRx. |
| RxJS | Reactive async | latest | Apache-2.0 | Approved. |

### Frontend tooling (shared)

| Tool | Purpose | License | Reason |
|---|---|---|---|
| ESLint + Prettier | Lint/format | MIT | Approved. |
| Jasmine + Karma + karma-coverage | Unit tests + coverage | MIT | Approved; 80% gate. |
| Playwright | Web E2E tests | Apache-2.0 | Approved. |

### Database & Infrastructure

| Item | Purpose | License / Service | Reason |
|---|---|---|---|
| PostgreSQL | Primary database | PostgreSQL License | Approved; only backend connects. |
| Neon | Managed PostgreSQL (prod) | Managed service (free tier) | Serverless Postgres host. |
| Cloudflare Pages | Web app hosting | Managed service | Static Angular build on CDN. |
| Render | Backend API host | Managed service ($7/mo Starter) | Confirmed; always-on, predictable cost, native Python. |
| Cloudflare R2 | Object storage (files/photos) | Managed service | Receipts, RC card, license docs. |
| GitHub | Source + CI/CD | Managed service | Repo and pipelines. |
| Google Play Store | Mobile distribution | Managed service | Android app delivery. |

---

## Architectural Decisions

| Date | Decision | Status | Notes |
|---|---|---|---|
| 2026-10-09 | **ORM = Async SQLAlchemy 2.0** | Confirmed | Chosen by user for PostgreSQL data access. |
| 2026-10-09 | **Migrations = Alembic** | Confirmed | Standard async-SQLAlchemy companion; version-controlled schema. |
| 2026-10-09 | Trip approval is single-level | Confirmed | Any admin or super-admin approves/rejects. |
| 2026-10-09 | Get Quotations merged into Trip Planner | Confirmed | One pre-trip module for planning + quotation. |
| 2026-10-09 | Multi-tenancy = shared DB + `tenant_id` (row-level) | Confirmed | Simplest isolation for the target scale. |
| 2026-10-09 | Data retention = permanent (no auto-purge) | Confirmed | Prefer soft-delete over hard delete. |
| 2026-10-09 | Currency = INR; GST breakdown configurable per record | Confirmed | Used when needed. |
| 2026-10-09 | Connectivity = online-only for v1 | Confirmed | Offline deferred to later release. |
| 2026-10-09 | No Docker for local development | Confirmed | Run apps directly; managed services for deploy only. |
| 2026-10-09 | Auth tokens = JWT access + refresh; 15 min access / 7 day refresh | Confirmed | Short-lived access limits risk; refresh keeps users logged in. |
| 2026-10-09 | Backend API host = Render | Confirmed | Flat $7/mo always-on, predictable cost, free tier for dev, native Python; fits Neon + R2. |
| 2026-10-09 | Backend API host = Render | Confirmed | Flat $7/mo always-on, predictable cost, free tier for dev, native Python; fits Neon + R2. |
| 2026-10-10 | UI design system defined (naval blue 🟦#1565C0, amber 🟨#FFA000, success 🟩#2E7D32, error 🟥#D32F2F, Roboto, light-only, rounded/friendly, vehicle loader) | Confirmed | See docs/ui_design_system_and_frontend_conventions.md. |
| 2026-10-10 | Frontend pattern = feature-based, standalone components, smart/dumb split, services + Signals | Confirmed | Same across all three apps; testable to 80% gate. |
| 2026-10-10 | Formats = Indian phone (6–9, 10-digit), Aadhaar 12-digit (stored encrypted, masked display), vehicle no., DD/MM/YYYY display + IST (store UTC), ₹ Indian grouping, password min 6 (no complexity) | Confirmed | See docs/validation_and_formats.md. |
| 2026-10-10 | API base URL in UI `environment.ts`; endpoint paths in shared code constants; backend endpoints in code, secrets in env | Confirmed | See docs/configuration_and_environments.md. |
| 2026-10-10 | Added coding standards, navigation map, config/env, API integration + error-code catalog docs | Confirmed | Reduces cross-developer ambiguity. |
| 2026-10-10 | Demo `staticData` flag (frontend-only, build-time, all 3 apps): true = local dummy JSON with in-memory writes + any-credentials login; false = real API | Confirmed | See docs/demo_static_data_mode.md. |
| 2026-10-10 | Revenue model = fare on Trip (`billed_amount` + optional GST + `advance_paid` + auto balance + `payment_status`); optional `estimated_fare` on Trip Planner; all roles can enter fare; **trip profit = fare - trip expenses** (service costs excluded); profit hidden from driver, reports super-admin only | Confirmed | Closes the profit-without-revenue gap. |

### Pending decisions
- **Logo** — real logo to replace the placeholder (user to provide).
# Coding Standards & Conventions
## Fleet Management System

| Field | Value |
|---|---|
| Version | 1.0 (Draft) |
| Date | 2026-10-10 |
| Status | Draft — ready for review |
| Related | docs/architecture_specification.md, docs/ui_design_system_and_frontend_conventions.md, docs/PROJECT-LOG.md |

> Goal: all developers write code the same way, so any file looks like one author wrote it. Linters/formatters enforce most of this automatically.

---

## 1. Shared Principles
- Keep functions small and single-purpose; prefer clear names over comments.
- No dead code, no commented-out blocks, no `console.log`/`print` debugging left in.
- Comment only to explain **why**, never **what** (one short line when needed).
- No secrets in code; read config from environment.
- Business logic lives in services/repositories, not in controllers/components.

---

## 2. Backend (Python 3.13 / FastAPI)

### 2.1 Tooling (enforced in CI)
| Tool | Purpose |
|---|---|
| Ruff | Lint + format (line length 100). |
| mypy | Static type checking (strict, where practical). |
| pytest + coverage | Tests, 80% gate. |
| pre-commit | Runs Ruff/mypy before commit. |

### 2.2 Naming
| Thing | Convention | Example |
|---|---|---|
| Modules/files | snake_case | `trip_service.py` |
| Functions/vars | snake_case | `get_trip_by_id` |
| Classes | PascalCase | `TripService` |
| Constants | UPPER_SNAKE | `ACCESS_TOKEN_TTL` |
| Pydantic schemas | PascalCase + suffix | `TripCreate`, `TripRead` |
| SQLAlchemy models | PascalCase singular | `Trip`, `TripExpense` |

### 2.3 Rules
- **Type hints everywhere** (params + returns); mypy must pass.
- **`async`** for all I/O (DB, HTTP); never block the event loop.
- Routers: validate + authorize + call a service; **no business logic or raw SQL**.
- Services: orchestrate use cases, own transactions, raise typed domain errors.
- Repositories: all DB access; aggregation/calculation in SQL; always tenant-filtered.
- Pydantic v2 for all request/response DTOs; never return ORM models directly.
- Imports: standard lib → third-party → local, each group sorted (Ruff isort).
- Errors: raise typed exceptions mapped to HTTP codes centrally; never `except: pass`.

### 2.4 Folder layout (per architecture doc)
```
app/
  core/          # config, db session, security, deps
  routers/       # HTTP layer
  services/      # use-case logic
  repositories/  # data access
  models/        # SQLAlchemy
  schemas/       # Pydantic DTOs
```

---

## 3. Frontend (Angular 20 / Ionic 8 / TypeScript)

### 3.1 Tooling (enforced in CI)
| Tool | Purpose |
|---|---|
| ESLint | Lint. |
| Prettier | Format (single source of style). |
| Jasmine + Karma + karma-coverage | Unit tests, 80% gate. |

### 3.2 Naming
| Thing | Convention | Example |
|---|---|---|
| Files | kebab-case + role suffix | `trip-list.page.ts`, `trip-card.component.ts`, `trip.service.ts`, `trip.model.ts` |
| Components/classes | PascalCase | `TripListPage`, `TripCardComponent` |
| Variables/functions | camelCase | `loadTrips()` |
| Constants | UPPER_SNAKE | `API_BASE_URL` |
| Observables | `$` suffix | `trips$` |
| Signals | plain noun | `trips`, `isLoading` |

### 3.3 Rules
- **Standalone components** only (no NgModules).
- **Angular Signals** for state; RxJS for streams — no NgRx or other state lib.
- **Smart/dumb split:** container pages fetch data + hold state; presentational components use `@Input()`/`@Output()` only, no API calls.
- Components never call HTTP directly — go through a feature `*.service.ts` that uses the shared API client.
- No `any` — type everything; import DTOs from `frontend/shared`.
- Unsubscribe/clean up (prefer `async` pipe / `toSignal` / `takeUntilDestroyed`).
- Follow the UI design system doc for styling, colors, components, and spacing.
- **UI-kit boundary:** Material/AG Grid/Bootstrap = web app only; Ionic = mobile only.

### 3.4 Folder layout (per app)
```
src/app/
  core/                # auth, http interceptor, guards, config
  shared/              # reusable presentational components, pipes, directives
  features/<feature>/
    pages/             # smart/container
    components/        # presentational
    services/          # feature API service
    models/             # view models
```

---

## 4. Testing Conventions
| Layer | Naming / Location |
|---|---|
| Backend | `tests/` mirroring `app/`; files `test_<unit>.py`; functions `test_<behavior>`. |
| Frontend | co-located `*.spec.ts` next to the file under test. |
| Style | Arrange-Act-Assert; one behavior per test; meaningful assertions (no coverage padding). |
| Mocks | Mock external boundaries (repos, HTTP, storage client); never hit real network/DB in unit tests. |

---

## 5. What CI will reject
- Lint/format/type errors (Ruff, mypy, ESLint, Prettier).
- Coverage below 80% line **or** branch.
- A UI kit used in the wrong app, or a non-approved dependency.
- Business logic in routers/components, or raw SQL outside repositories.
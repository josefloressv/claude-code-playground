# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app locally

**Backend** — from `Backend/`:
```bash
make start    # Docker: API on :8000, PostgreSQL on :5432
make migrate  # run Alembic migrations (first run or after schema change)
make seed     # load sample data
```

**Frontend** — from `Frontend/`:
```bash
yarn install  # only needed once
yarn dev      # Next.js on :3000 (falls back to :3001 if busy)
```

The frontend fetches `http://localhost:8000` — both must be running together.

---

## Backend (`/Backend`)

**Stack:** Python 3.11, FastAPI, SQLAlchemy 2.0, Alembic, PostgreSQL, Docker, `uv`

Architecture: `main.py` (routes + `Depends()` injection) → `services/` (business logic) → `models/` (ORM) → `db/base.py` (engine + session)

All models inherit `BaseModel`: `id`, `created_at`, `updated_at`, `deleted_at`. Always filter soft-deletes with `Model.deleted_at.is_(None)`.

**Run tests:**
```bash
docker-compose exec api bash -c "cd /app && uv run pytest app/test_main.py -v"
# single test:
docker-compose exec api bash -c "cd /app && uv run pytest app/test_main.py::TestCoursesEndpoints::test_get_all_courses_success -v"
```

Tests use `fastapi.testclient.TestClient` + `app.dependency_overrides` — no real database needed.

**Code conventions (from Cursor rules):**
- `def` for sync, `async def` for async I/O
- Early returns / guard clauses; avoid nested conditionals
- `HTTPException` for expected errors; services return `None` on not-found, routes raise 404
- Type hints on all signatures

---

## Frontend (`/Frontend`)

**Stack:** Next.js 15, React 19, TypeScript, SASS, CSS Modules, Vitest, Testing Library

```bash
yarn lint   # ESLint
yarn test   # Vitest (watch)
yarn test src/components/Course/__test__/Course.test.tsx  # single file
```

Path alias `@` → `./src`. Pages live in `src/app/` (App Router). Each component has a co-located `.module.scss` and `.test.tsx`.

---

## Mobile

**Android** (`Mobile/PlatziFlixAndroid`): Kotlin · Jetpack Compose · MVVM · Hilt · Retrofit. Build via Android Studio or `./gradlew assembleDebug`.

**iOS** (`Mobile/PlatziFlixiOS`): Swift · SwiftUI · Clean Architecture (Data / Domain / Presentation). Build via Xcode.

---

## API contract

Canonical source: `Backend/specs/00_contracts.md`

| Endpoint | Returns |
|---|---|
| `GET /courses` | `[{id, name, description, thumbnail, slug}]` |
| `GET /courses/{slug}` | `{…, teacher_id[], classes[{id, name, description, slug}]}` |
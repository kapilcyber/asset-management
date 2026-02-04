# ASSET-MANAGER (ITSM Asset Management Platform)

Full‑stack IT Service Management (ITSM) + Asset Management platform:

- **Frontend**: Next.js 14 (`frontend/`) on `http://localhost:3000`
- **Backend API**: FastAPI + Async SQLAlchemy (`backend/`) on `http://localhost:8000`
- **Database**: PostgreSQL (Docker Compose provided)

The platform covers asset inventory, procurement workflow, helpdesk/ticketing, offboarding, audit logs, and dashboards with role-based access control.

## Key features

- **Role-based access control (RBAC)**: multiple roles (Admin, IT, Inventory/Asset Manager, Finance, Manager, Employee)
- **Asset management**: create/update assets, assignment, lifecycle status, renewals
- **Procurement workflow**: multi-stage approvals (Manager → IT → Inventory/Procurement → Finance), PO upload/extraction, delivery + QC
- **Helpdesk**: incident tickets linked to assets + diagnostics
- **Offboarding**: exit workflow with asset reclamation / BYOD processing
- **Audit**: audit log APIs and dashboard views

## Repo structure

- `frontend/`: Next.js app (pages, components, contexts)
- `backend/app/`: FastAPI app (routers under `/api/v1`)
- `backend/scripts/`: one-off utilities (DB setup, seeders, diagnostics)
- `DATABASE_OPTIMIZATIONS.md`: DB indexing/pagination notes
- `ITSM_MASTER.md`: product/feature overview & architecture notes

## Quickstart (Docker: Postgres + Backend) + local Frontend

### 1) Start Postgres + API

From `backend/`:

```bash
docker compose up --build
```

This starts:

- **Postgres**: host port `5433` → container `5432`
  - DB: `ITSM`
  - User: `postgres`
  - Password: `1234`
- **Backend API**: `http://localhost:8000`
  - Swagger docs: `http://localhost:8000/docs`
  - Health: `http://localhost:8000/health`
  - DB health: `http://localhost:8000/health/db`

### 2) Start the frontend (separate terminal)

From `frontend/`:

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

## Quickstart (Local dev without Docker)

### Backend

Requirements:

- Python **3.9+**
- PostgreSQL 13+ (local install) or run Postgres via Docker only

From `backend/`:

```bash
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# macOS/Linux:
# source .venv/bin/activate

pip install -r requirements.txt

# Example (adjust to your DB):
setx DATABASE_URL "postgresql+asyncpg://postgres:postgres@localhost:5432/itsm"
setx DEBUG "true"

uvicorn app.asgi:app --reload --host 0.0.0.0 --port 8000
```

### Frontend

From `frontend/`:

```bash
npm install
npm run dev
```

## Configuration

### Frontend → Backend URL

Frontend API base URL is controlled by:

- **`NEXT_PUBLIC_API_URL`** (optional)
  - Default: `http://127.0.0.1:8000`
  - The frontend calls the versioned API under `/api/v1`

Example:

```bash
setx NEXT_PUBLIC_API_URL "http://127.0.0.1:8000"
```

### Backend environment variables

Common env vars used in the backend:

- **`DATABASE_URL`**: SQLAlchemy URL (async dialect recommended: `postgresql+asyncpg://...`)
- **`DEBUG`**: `true/false` (enables more verbose error responses)
- **`SECRET_KEY`** / JWT settings: see `backend/app/config/settings.py` (defaults should be changed for production)
- **`COLLECT_API_TOKEN`**: token for `/api/v1/collect` (if enabled/required in your setup)

## Database setup & seed data (optional)

There are helper scripts under `backend/scripts/`. Run them from the `backend/` directory so imports resolve correctly.

### Create schemas + tables

```bash
python scripts/setup_database.py
```

### Populate mock assets (50 assets)

```bash
python scripts/populate_mock_data.py
```

### Seed stock / renewals / tickets

```bash
python scripts/seed_stock.py
python scripts/seed_renewals.py
python scripts/seed_tickets.py
```

### Reset common dev passwords (if users already exist)

If your DB already contains the dev users, you can reset passwords via:

```bash
python scripts/reset_test_passwords.py
```

This script targets emails like `admin@itsm.com`, `it_manager@itsm.com`, `it@test.com`, `asset@test.com` and sets the password to `password123` (dev only).

## API notes

- Base API prefix: **`/api/v1`**
- Interactive docs: **`/docs`**
- CORS is configured for local frontend origins (e.g. `http://localhost:3000`)

## Troubleshooting

- **Backend can’t connect to DB**:
  - Confirm Postgres is reachable and `DATABASE_URL` points to the right host/port.
  - If you use Docker Compose, the backend container connects to host `db:5432` (not `localhost`).
- **Frontend calls wrong API URL**:
  - Set `NEXT_PUBLIC_API_URL` and restart `npm run dev`.
- **Need a clean DB**:
  - Stop compose and remove volumes: `docker compose down -v` (from `backend/`)


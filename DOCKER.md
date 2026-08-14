# Vantage Platform — Docker Deployment Guide

Each split project is **independently deployable** with its own `Dockerfile` and `docker-compose.yml`. Use `vantage-platform` to run the full stack with a single command.

## Full stack (recommended)

```powershell
cd D:\root\projects\vantage-platform
copy .env.example .env
docker compose up -d --build
docker compose --profile migrate-all run --rm migrate-migration-db
```

All services start by default — no Compose profiles required for the main stack.

| Service | URL |
|---------|-----|
| harness-core API | http://localhost:50223/v1 |
| harness-core dashboard (bundled UI) | http://localhost:50223/dashboard |
| telemetry-service API | http://localhost:50224 |
| telemetry-service dashboard (bundled UI) | http://localhost:50224/dashboard |
| harness-ui (standalone nginx) | http://localhost:50225 |
| PostgreSQL | localhost:5433 |

Optional: `telemetry-ui` nginx on `:50226` — `docker compose --profile standalone-ui up -d telemetry-ui`

### Run a subset of services

```powershell
# Backend only (no harness-ui container)
docker compose up -d db telemetry-service harness-core

# Optional separate telemetry nginx (:50226) — not needed for normal use
docker compose --profile standalone-ui up -d telemetry-ui

# Harness shell (embeds telemetry from :50224/dashboard)
docker compose up -d harness-ui
```

---

## Per-project standalone deploy

Deploy any component alone when you do not need the full platform compose file.

| Project | Command | URL |
|---------|---------|-----|
| **vantage-harness-db** | `docker compose up -d` | PostgreSQL `:5432` |
| **vantage-telemetry-service** | `docker compose up -d --build` | `:50224` |
| **vantage-harness-core** | `docker compose up -d --build` | `:50223` |
| **vantage-telemetry-ui** | `docker compose up -d --build` | `:50226` |
| **vantage-harness-ui** | `docker compose up -d --build` | `:50225` |

### 1. Database only

```powershell
cd D:\root\projects\vantage-harness-db
copy .env.example .env
docker compose up -d
docker compose --profile migrate run --rm migrate
docker compose --profile migrate-all run --rm migrate-migration-db
```

### 2. Telemetry service only

Requires PostgreSQL reachable via `DATABASE_URL`.

```powershell
cd D:\root\projects\vantage-telemetry-service
copy .env.example .env
docker compose up -d --build
```

Bundled UI: http://localhost:50224/dashboard

### 3. Harness core only

Requires PostgreSQL + telemetry-service + Ollama on host.

```powershell
cd D:\root\projects\vantage-harness-core
copy .env.example .env
docker compose up -d --build
```

API: http://localhost:50223/v1

### 4. Telemetry UI only (separate static site)

```powershell
cd D:\root\projects\vantage-telemetry-ui
copy .env.example .env
docker compose up -d --build
```

UI: http://localhost:50226 → API via `VANTAGE_TELEMETRY_URL`

### 5. Harness UI only (separate static site)

```powershell
cd D:\root\projects\vantage-harness-ui
copy .env.example .env
docker compose up -d --build
```

UI: http://localhost:50225 → harness-core + embedded telemetry iframe

---

## UI deployment modes

| Mode | Description |
|------|-------------|
| **Bundled** | Static assets baked into `telemetry-service` / `harness-core` images (`/dashboard`) |
| **Separate** | `vantage-telemetry-ui` / `vantage-harness-ui` nginx containers on `:50226` / `:50225` |

Runtime API URLs are injected via `docker/entrypoint.sh` → `static/js/runtime-config.js`. Separate UI containers use **browser-facing** `localhost` URLs, not Docker internal hostnames.

---

## Prerequisites

- **Ollama** running on host (`11434`) for harness-core local inference
- Cloud keys optional (`OPENROUTER_API_KEY`, etc.) in `.env`
- On Windows/Mac Docker Desktop, `host.docker.internal` reaches the host from containers

---

## Production notes

- Change `TELEMETRY_INTERNAL_KEY` and `TELEMETRY_BRIDGE_JWT_SECRET` in every service `.env`
- Use external managed PostgreSQL — set `DATABASE_URL` and skip the `db` service
- Put a reverse proxy (Traefik/nginx) in front for TLS and single-origin cookies
- Original monolith (`vantage-harness` on `:50123`) remains independent during migration

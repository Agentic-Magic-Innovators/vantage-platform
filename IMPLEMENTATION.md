# Vantage Split Stack — Implementation Summary

The monolith at `vantage-harness` (port **50123**) is **unchanged**. New projects run on migration ports **50223–50224**.

## Projects

| Directory | Port | Role |
|-----------|------|------|
| [vantage-harness-db](../vantage-harness-db) | 5432 / 5433 | PostgreSQL migrations |
| [vantage-telemetry-service](../vantage-telemetry-service) | 50224 | Telemetry ingest + `/metrics` |
| [vantage-harness-core](../vantage-harness-core) | 50223 | OpenAI gateway + routing |
| [vantage-telemetry-ui](../vantage-telemetry-ui) | (static) | Standalone telemetry SPA |
| [vantage-harness-ui](../vantage-harness-ui) | (static) | Harness admin + embedded telemetry |
| [vantage-platform](./) | — | Docker Compose for full stack |

## Quick start (local)

```powershell
# 1. Database
cd D:\root\projects\vantage-harness-db
docker compose up -d
pip install asyncpg
python scripts\apply_migrations.py
python scripts\apply_migrations.py postgresql://vantage:vantage@localhost:5432/vantage_migration

# 2. Telemetry service
cd D:\root\projects\vantage-telemetry-service
$env:DATABASE_URL = "postgresql://vantage:vantage@localhost:5432/vantage_migration"
$env:TELEMETRY_INTERNAL_KEY = "dev-internal-key"
$env:TELEMETRY_BRIDGE_JWT_SECRET = "dev-bridge-secret"
python -m app.main

# 3. Harness core (new terminal)
cd D:\root\projects\vantage-harness-core
$env:VANTAGE_TELEMETRY_URL = "http://localhost:50224"
$env:TELEMETRY_INTERNAL_KEY = "dev-internal-key"
$env:DATABASE_URL = "postgresql://vantage:vantage@localhost:5432/vantage_migration"
python -m app.request_gateway
```

## URLs

- Harness API (Cursor/Continue): http://localhost:50223/v1
- Harness dashboard: http://localhost:50223/dashboard
- Telemetry API: http://localhost:50224/v1/telemetry
- Telemetry dashboard: http://localhost:50224/dashboard

## MCP config (`~/.vantage/config.json`)

```json
{
  "telemetry_url": "http://localhost:50224",
  "harness_url": "http://localhost:50223",
  "api_key": "vh_live_..."
}
```

## Docker (all services)

```powershell
cd D:\root\projects\vantage-platform
docker compose up -d --build
```

## Architecture notes

- **Only telemetry-service writes `metrics`** (MCP + harness internal events).
- **Harness-core** sends inference metrics via `TelemetryClient` → `POST /v1/internal/events`.
- **Deprecated proxy**: `POST http://localhost:50223/v1/telemetry` forwards to telemetry-service.
- **Embedded telemetry**: harness-ui requests bridge JWT → iframe to telemetry-ui with `?bridge=`.

See [docs/TELEMETRY_ARCHITECTURE.md](./docs/TELEMETRY_ARCHITECTURE.md) and [docs/HARNESS_ARCHITECTURE.md](./docs/HARNESS_ARCHITECTURE.md) for full architecture diagrams and data flows.

See [vantage-harness/docs/vantage_split_implementation_plan.md](../vantage-harness/docs/vantage_split_implementation_plan.md) for full migration plan.

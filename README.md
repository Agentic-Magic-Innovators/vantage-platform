# Vantage Platform (split stack)

Orchestrates all post-split services on migration ports **50223–50226**. The original monolith on **50123** is untouched.

## Quick start

```powershell
cd D:\root\projects\vantage-platform
copy .env.example .env
docker compose up -d --build
docker compose --profile migrate-all run --rm migrate-migration-db
```

| Service | URL |
|---------|-----|
| harness-core API | http://localhost:50223/v1 |
| harness-core dashboard | http://localhost:50223/dashboard |
| telemetry-service | http://localhost:50224 |
| telemetry-service dashboard | http://localhost:50224/dashboard |
| harness-ui (standalone) | http://localhost:50225 |
| telemetry-ui (optional) | http://localhost:50226 — `--profile standalone-ui` only |
| PostgreSQL | localhost:5433 |

## Documentation

- [docs/TELEMETRY_ARCHITECTURE.md](./docs/TELEMETRY_ARCHITECTURE.md) — telemetry collectors, ingest, storage, dashboards
- [docs/HARNESS_ARCHITECTURE.md](./docs/HARNESS_ARCHITECTURE.md) — gateway, routing, auth, caching, observability
- [DOCKER.md](./DOCKER.md) — full Docker guide (standalone per-project deploy + full stack)
- [IMPLEMENTATION.md](./IMPLEMENTATION.md) — architecture and local dev without Docker

## Deploy one component only

Each sibling repo has its own `Dockerfile` and `docker-compose.yml`:

- `../vantage-harness-db`
- `../vantage-telemetry-service`
- `../vantage-harness-core`
- `../vantage-telemetry-ui`
- `../vantage-harness-ui`

See [DOCKER.md](./DOCKER.md) for commands.

# Energy Platform

The Energy Platform is a device-agnostic foundation for ingesting, storing, and serving energy system data across heterogeneous devices and integrations. Phase 1 establishes the core platform primitives needed to onboard assets, capture raw telemetry, normalize operational measurements, and surface tablet-friendly dashboards while preserving flexibility for future device adapters and discovery workflows.

## Project Overview

Phase 1 delivers the initial operational backbone of the platform:

- **Device registry** for managing energy assets in a central catalog.
- **Telemetry ingestion** through raw-first data capture and normalized time-series storage.
- **Recommendations foundation** through analytics-ready telemetry persistence and aggregation.
- **Tablet dashboards** via a responsive frontend scaffold optimized for operational visibility.

### Architecture philosophy

The platform follows three core design principles:

- **Device-agnostic**: the data model is built around assets and capabilities rather than vendor-specific device definitions.
- **Raw-first**: source payloads are stored before normalization to support replay, auditing, and iterative parser improvements.
- **Adapter-based**: integrations can be added incrementally through adapters without reshaping the platform core.

## Quick Start

### Prerequisites

- Docker
- Docker Compose
- Node.js 20+

### Clone and setup

1. Clone the repository.
2. Copy [` .env.example`](.env.example) to `.env` for local configuration:

   ```bash
   cp .env.example .env
   ```

### Start services

Run the full local stack with either [`compose.linux.full.yml`](compose.linux.full.yml) or [`compose.windows.full.yml`](compose.windows.full.yml):

```bash
docker-compose -f compose.linux.full.yml up -d
```

```bash
docker-compose -f compose.windows.full.yml up -d
```

### Access points

- Frontend: http://localhost:3001
- Backend: http://localhost:3000
- API health: http://localhost:3000/health

Additional setup notes are available in [`docs/SETUP.md`](docs/SETUP.md).

## Project Structure

- [`backend/`](backend/) — Node.js/Express API for health checks, metrics, and future ingestion and recommendation services.
- [`frontend/`](frontend/) — Next.js 16 tablet-first dashboard and PWA shell.
- [`migrations/`](migrations/) — versioned SQL schema and TimescaleDB optimization scripts.
- [`docs/`](docs/) — supporting setup and implementation documentation.

### Key files

- [`compose.linux.full.yml`](compose.linux.full.yml) — local orchestration for TimescaleDB, collector, backend, and frontend services with local network.
- [`compose.windows.full.yml`](compose.windows.full.yml) — local orchestration for TimescaleDB, collector, backend, and frontend services. NOT recommended, auto discovery does not work.
- [` .env.example`](.env.example) — shared environment variable template for local development.

## Technology Stack

### Backend

- Node.js
- Express
- PostgreSQL / TimescaleDB
- Pino logging

### Frontend

- Next.js 16.0.10
- React
- TypeScript
- Tailwind CSS
- Progressive Web App (PWA) capabilities

### Infrastructure

- Docker Compose
- TimescaleDB

## Development Workflow

### Run the full stack with Docker Compose

Use [`docker-compose.yml`](docker-compose.yml) to start all services:

```bash
docker-compose up -d
```

Stop the stack when needed:

```bash
docker-compose down
```

### Viewing logs

Inspect service logs with:

```bash
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f timescaledb
docker-compose logs -f redis
```

Or replace the service name as needed:

```bash
docker-compose logs -f [service]
```

## API Endpoints (Phase 1)

Phase 1 currently exposes operational endpoints:

- `GET /health` — service health and dependency status.
- `GET /metrics` — Prometheus-format metrics for monitoring and observability.

Relevant route implementations:

- [`backend/src/routes/health.js`](backend/src/routes/health.js)
- [`backend/src/routes/metrics.js`](backend/src/routes/metrics.js)

## Database Schema

The platform data model is defined in [`migrations/`](migrations/), with an emphasis on separating registry data, immutable raw ingestion, and normalized telemetry.

### Key tables

- `site` — site-level deployment context including timezone and currency.
- `asset` — registry of managed energy assets.
- `asset_identity` — durable identities per asset such as serials, mqtt topics, or mdns names.
- `capability_definition` — global catalog of capability types.
- `asset_capability` — attached capabilities and configuration per asset.
- `discovery_record` — detected devices and onboarding evidence.
- `mapping_profile` — codeless source-to-normalized field mappings.
- `raw_event` — immutable raw source payloads for replay and auditing.
- `telemetry_sample` — normalized measurement samples for analytics and dashboards.
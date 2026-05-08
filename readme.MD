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

### Backend development

From [`backend/`](backend/):

```bash
npm install
npm start
```

The backend currently exposes operational endpoints and provides the scaffold for ingestion, recommendations, and device adapter services. See [`backend/README.md`](backend/README.md).

### Frontend development

From [`frontend/`](frontend/):

```bash
npm install
npm run dev
```

The frontend development server runs on port `3000` inside the app directory; in Docker it is mapped to host port `3001`. See [`frontend/README.md`](frontend/README.md).

### Database migrations

Schema initialization is driven by the SQL files in [`migrations/`](migrations/), which are mounted into the TimescaleDB container through [`docker-compose.yml`](docker-compose.yml). On a fresh database volume, these initialization scripts are applied automatically by the container entrypoint during first boot.

Primary migration files:

- [`migrations/V1__initial_schema.sql`](migrations/V1__initial_schema.sql)
- [`migrations/V2__hypertables_and_compression.sql`](migrations/V2__hypertables_and_compression.sql)

Implementation details and manual execution notes are documented in [`docs/SETUP.md`](docs/SETUP.md).

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

### Time-series optimization

[`migrations/V2__hypertables_and_compression.sql`](migrations/V2__hypertables_and_compression.sql) converts [`raw_event`](migrations/V1__initial_schema.sql) and [`telemetry_sample`](migrations/V1__initial_schema.sql) into TimescaleDB hypertables and applies:

- compression policies for older chunks,
- retention policies for long-term storage management,
- and a continuous aggregate for 5-minute telemetry averages.

This provides a solid Phase 1 foundation for scalable time-series ingestion and analytics.

## Next Steps

Phase 1 establishes the core platform foundation and is ready for the next layer of implementation work, including device adapters, discovery flows, richer ingestion pipelines, and recommendation logic.

For the broader roadmap and feature evolution, see [`energy_platform_user_stories.html`](energy_platform_user_stories.html).

## Current User Story Coverage

This repository now contains a mix of completed scaffolds, partial vertical slices, and placeholder integrations aligned to the ordered board in [`energy_platform_user_stories.html`](energy_platform_user_stories.html).

### Largely covered foundation stories

- `US-001` — local Docker-based development stack in [`docker-compose.yml`](docker-compose.yml), [`backend/Dockerfile`](backend/Dockerfile), and [`frontend/Dockerfile`](frontend/Dockerfile)
- `US-003` — backend scaffold, health, and metrics in [`backend/src/index.js`](backend/src/index.js), [`backend/src/routes/health.js`](backend/src/routes/health.js), and [`backend/src/routes/metrics.js`](backend/src/routes/metrics.js)
- `US-004` — tablet-first frontend shell and route structure in [`frontend/app/layout.tsx`](frontend/app/layout.tsx) and [`frontend/app/`](frontend/app/page.tsx)

### Partially implemented backend platform stories

- `US-002` — capability-first schema baseline in [`migrations/V1__initial_schema.sql`](migrations/V1__initial_schema.sql)
- `US-005` / `US-006` — asset and capability APIs in [`backend/src/routes/assets.js`](backend/src/routes/assets.js)
- `US-007` / `US-008` — discovery and combined device overview in [`backend/src/routes/discovery.js`](backend/src/routes/discovery.js)
- `US-009` / `US-010` / `US-011` — raw ingestion, telemetry write path, and telemetry read APIs in [`backend/src/routes/telemetry.js`](backend/src/routes/telemetry.js)
- `US-012`, `US-014`, `US-015` — MQTT discovery inference and onboarding endpoints in [`backend/src/routes/discovery.js`](backend/src/routes/discovery.js)
- `US-016` / `US-017` — adapter loading, mapping profiles, and dry-run support in [`backend/src/adapters/registry.js`](backend/src/adapters/registry.js) and [`backend/src/routes/adapters.js`](backend/src/routes/adapters.js)

### Partially implemented adapter stories

- `US-018` — P1 smart meter adapter in [`backend/src/adapters/p1SmartMeterAdapter.js`](backend/src/adapters/p1SmartMeterAdapter.js)
- `US-019` — solar inverter adapter in [`backend/src/adapters/solarInverterAdapter.js`](backend/src/adapters/solarInverterAdapter.js)
- `US-020` — battery inverter adapter in [`backend/src/adapters/batteryInverterAdapter.js`](backend/src/adapters/batteryInverterAdapter.js)
- `US-021` — EV charger adapter in [`backend/src/adapters/evChargerAdapter.js`](backend/src/adapters/evChargerAdapter.js)
- `US-022` — climate adapter in [`backend/src/adapters/climateAdapter.js`](backend/src/adapters/climateAdapter.js)
- `US-023` — generic Modbus meter scaffold and sample config in [`backend/src/adapters/modbusPowerMeterAdapter.js`](backend/src/adapters/modbusPowerMeterAdapter.js) and [`docs/samples/modbus-power-meter.sample.json`](docs/samples/modbus-power-meter.sample.json)

### Partially implemented data/recommendation/dashboard stories

- `US-024` / `US-025` — placeholder external data snapshot APIs for price and weather in [`backend/src/routes/externalData.js`](backend/src/routes/externalData.js)
- `US-026` / `US-027` — recommendation compute, current state, history, and threshold settings in [`backend/src/routes/recommendation.js`](backend/src/routes/recommendation.js)
- `US-028` / `US-031` / `US-032` — live overview, prices, weather, and diagnostics UI slices in [`frontend/app/page.tsx`](frontend/app/page.tsx), [`frontend/app/prices/page.tsx`](frontend/app/prices/page.tsx), [`frontend/app/weather/page.tsx`](frontend/app/weather/page.tsx), and [`frontend/app/settings/page.tsx`](frontend/app/settings/page.tsx), now including time-range-driven telemetry history and dynamic asset discovery for the dashboard

### Partially implemented hardening stories

- `US-035` / `US-036` — stale-data exposure and adapter diagnostics counters in [`backend/src/repositories/assetsRepository.js`](backend/src/repositories/assetsRepository.js), [`backend/src/adapters/registry.js`](backend/src/adapters/registry.js), and [`backend/src/routes/diagnostics.js`](backend/src/routes/diagnostics.js)
- `US-037` / `US-038` — replay jobs and request tracing in [`backend/src/routes/replay.js`](backend/src/routes/replay.js), [`backend/src/repositories/replayRepository.js`](backend/src/repositories/replayRepository.js), [`backend/src/utils/logger.js`](backend/src/utils/logger.js), and [`backend/src/middleware/errorHandler.js`](backend/src/middleware/errorHandler.js)

### Most valuable next build steps

1. Add durable scheduling/locking for Nord Pool and Open-Meteo sync jobs beyond in-process intervals.
2. Deepen the live overview into the full energy-flow dashboard from `US-028` / `US-029`.
   - The dashboard now uses time-range-driven telemetry series, dynamic asset discovery, and explicit energy-flow visualization, but it still needs richer null-gap handling, better device-role inference, and polished interaction.
3. Turn adapter scaffolds into protocol-backed integrations with stronger normalization coverage.
4. Make replay, stale detection, and adapter fault handling durable rather than in-memory/runtime-local.
5. Add tests for the key backend routes, adapters, and recommendation logic.

## Contributing

Use the following guidelines when contributing to the project:

- Follow the existing structure and naming conventions used in [`backend/`](backend/) and [`frontend/`](frontend/).
- Keep code changes focused, readable, and aligned with the platform’s device-agnostic and adapter-based architecture.
- Run relevant local checks before opening a pull request, including application startup and any available linting or manual validation steps.
- Update documentation such as [`README.md`](README.md), [`backend/README.md`](backend/README.md), [`frontend/README.md`](frontend/README.md), or [`docs/SETUP.md`](docs/SETUP.md) when behavior or setup changes.
- Open pull requests with a clear description of scope, testing performed, and any migration or configuration impact.

For environment and infrastructure setup details, refer to [`docs/SETUP.md`](docs/SETUP.md).



#example way of changing database:
"C:\Program Files\Git\bin\bash.exe" -c 'PGPASSWORD=changeme docker exec -e PGPASSWORD=changeme energy-platform-timescaledb psql -U energy_user -d energy_platform -c "ALTER TABLE mapping_profile ADD COLUMN IF NOT EXISTS target_capability VARCHAR(64); ALTER TABLE mapping_profile ADD COLUMN IF NOT EXISTS target_field VARCHAR(128);"'



 curl -s "http://192.168.1.13:3000/api/v1/telemetry/latest?asset_ids=51&fields=generated_power_w,consumed_power_w&max_age_ms=60000"


 wat heb ik nodig voor live gang?

 toonbaar dashboard
 product wat ik kan shippen, dit betekent:
   in te richten met welk product dan ook, zonder problemen
   geen dubbele assets
   setup flow - mqtt, adres / location, energie contract
   ondubbelzinnige config.



te ondersteunen
 slimme thermostaat
 warmtepomp             v
 airco
 thuis batterij         v
 EV charger    
 solar panel inverter   v


ik moet device specs kunnen koppelen op basis van meta data in de devicespecs. kan een lijst zijn van woorden zoals:
 smart-gateways, dsmr

 of 

 solar

 of
 
 enphase


 als hij daar niet mee kan matchen (door te zoeken in de discovery records voor deze woorden), dan mag hij het proberen door naar (in het geval van esphome) te kijken naar de bestaande entities en devicespecs met de meeste zelfde fieldnames. kijk ook even wat je voor andere protocollen kan doen, bijvoorbeeld bij mqtt kan je de entities of velden ook al zien, en bij http kan je het proberen door html parsing te doen van tabellen.

 als hij daar niet mee kan matchen dan kunnen we nog proberen uit te zoeken wat elk van die (in het geval van esphome) entities zijn en dan proberen zelf een device spec te maken, custom, en die er automatisch aan te hangen. dat kan je ook voor de andere protocollen doen.

 en dit hoeft natuurlijk niet voor wled, of kpn, want dat zijn devices waar ik niet in geinteresseerd ben.
 de AI mag dit vervolgens allemaal gaan aanvullen, dan vul ik het systeem.




HomeWizard Energy – P1 Meter
Product: HomeWizard P1 Meter (HWE‑P1)

Ecosystem: HomeWizard Energy app + cloud; also supports Home Assistant via local API.

API: Yes, local JSON API at http://<ip>/api/v1/data (Poll‑based); you can also poll battery status via /api/batteries when it controls Plug‑in Batteries.

Remarks: Very popular in NL; often used as the “universal” P1 meter for many battery brands.

Plugwise – Smile P1
Product: Plugwise Smile P1

Ecosystem: Plugin‑based smart‑home stack; integrates well with Home Assistant and other local systems.

API: Yes, local API (HTTP/WebSocket) and documented REST‑style endpoints; more open‑oriented.

Remarks: Sold in NL, focused on local control and open standards.

Chargee – Sparky
Product: Chargee Sparky

Ecosystem: Sparky app and cloud (AI‑enhanced consumption/forecasting).

API/hook: Home Assistant integration exists; effectively acts as a control/logic layer with its own API‑style integration.

Remarks: Positioned as a “smart” P1 meter with cloud‑based analytics.

EnergyFlip
Product: EnergyFlip P1 meter

Ecosystem: Local‑heavy, with optional cloud dashboards.

API: Documented local API available for direct polling; can also integrate with Home Assistant.

Remarks: Popular in NL for people wanting local‑first, no‑abonnement dashboards.

Homey Energy Dongle
Product: Homey Energy Dongle (P1)

Ecosystem: tightly integrated with Homey, but also used by EcoFlow as an alternative to their own P1 meter.

API: via Homey flows/API; not a raw HTTP API per se, but still programmable from scripts.

Remarks: Very common in Homey‑centric Dutch homes.

EcoFlow P1 Meter
Product: EcoFlow P1 Meter

Ecosystem: EcoFlow app + their Stream/SolarFlow batteries.

API: Proprietary API to EcoFlow cloud; also exposes data via EcoFlow app and integrations.

Remarks: Used when you want to optimize EcoFlow batteries in NL.

Altilia P1 Meter
Product: Altilia P1 Meter

Ecosystem: Cloud‑based dashboard and analytics.

API: exposes data via cloud API; targets integrators/partners.

Remarks: Used in some pro/consumer setups for real‑time dashboards.



HomeWizard – Plug‑in Battery
Product: HomeWizard Plug‑in Battery

Ecosystem: Controlled via HomeWizard Energy app; uses HomeWizard P1 meter or 3‑IN‑1 meter for grid data.

API: P1 meter exposes /api/batteries (read‑only power, mode) and the P1 meter itself has a local REST API.

Remarks: Very popular in NL; effectively “API‑enabled” via HomeWizard P1 meter.

Indevolt – PowerFlex 2000 series
Product: Indevolt PowerFlex 2000, PowerFlex 3000

Ecosystem: app + cloud, but supports local API for Home Assistant integration.

API: documented local API endpoints for state of charge, charge/discharge power, and some control modes.

Remarks: Frequently recommended in NL P1 meter compatibility lists; very API‑friendly.

Marstek / Marstek Power
Product: Marstek plug‑in batteries (Europe‑oriented)

Ecosystem: own app + cloud; can be controlled via its own P1 meter (CT003) or via HomeWizard P1 (with limitation).

API: proprietary API exposed via their P1 meter and cloud; not fully open, but documented for some integrations.

Remarks: Sold by many Dutch installers and resellers.

Zendure – SolarFlow / Stream
Product: Zendure SolarFlow (wall battery), Zendure Stream (plug‑in battery)

Ecosystem: Zendure app + cloud; uses Zendure P1 meter (or sometimes HomeWizard P1) for grid data.

API: limited but documented API for grid/battery power; some Home Assistant integrations scrape or wrap it.

Remarks: Growing in NL; you often see them paired with HomeWizard P1 meters.

Anker – SOLIX (EU Thuisbatterijen)
Product: Anker SOLIX C800/C1200, SOLIX‑based home batteries

Ecosystem: Anker app and cloud; includes its own P1/PV meter as part of the system.

API: proprietary cloud API; third‑party integrations exist (Home Assistant, etc.) but the API is not fully public.

Remarks: Sold via Dutch retailers and installers; closed but still controllable via Home Assistant style bridges.

EcoFlow – DELTA Pro / Stream / SolarFlow
Product: EcoFlow Stream (plug‑in battery), DELTA Pro / EcoFlow wall batteries, SolarFlow

Ecosystem: EcoFlow app and cloud; can be controlled via EcoFlow P1 meter or Homey Energy Dongble.

API: Proprietary API (Partial documentation exists in community projects); Home Assistant integrations rely on reverse‑engineered API calls.

Remarks: Fast‑growing in NL; tightly closed but still accessible via wrapper scripts.

OnzeStroom / Other “slimme aansturing” vendors
Product: various branded batteries (often Indevolt, Marstek, Anker‑based)

Ecosystem: OnzeStroom platform + app; offers “open API” for partners.

API: REST API to control/monitor battery and grid behavior via their platform.

Remarks: Targets Dutch installers and EMS providers; you can integrate via their API instead of the vendor directly.

wij zijn stroomr. de onafhankelijk badass op de stroommarkt.

take back the power.



The main ones are:

Smart meter / main electricity meter
This is the foundation. It shows total import/export, current usage, and sometimes gas usage too.

Solar inverter
Tells the system how much solar power is being generated right now.

Home battery
Important for charging/discharging decisions, peak shaving, and using cheap or self-generated power at the right time.

EV charger / electric car
One of the biggest controllable loads in many homes. Great for charging when solar is high or tariffs are low.

Heat pump
Often a major consumer. Very relevant because heating can often be shifted slightly without losing comfort.

Boiler / electric water heater / heat pump boiler
Hot water heating is a good flexible load for using excess solar.

Air conditioning
Another large controllable load, especially in summer.

Washing machine, dryer, dishwasher
Useful because they can often run later in the day when electricity is cheaper or solar is available.

Induction cooktop / oven
Usually not very shiftable, but still useful to monitor because they create peaks.

Pool pump / sauna / jacuzzi / hot tub
Not present in every home, but very relevant if they are, because they can use a lot of power and are often schedulable.

Mechanical ventilation / HVAC fans
Smaller than heating, but still interesting in efficient homes.

Smart plugs / large standalone appliances
For things like freezers, dehumidifiers, portable heaters, or other appliances you want to monitor or switch.

Submeters / circuit clamps
These help measure big groups separately, like kitchen, heat pump, EV charger, or upstairs/downstairs.

Dynamic tariff data source
Not a physical device, but very important for decision-making if the home uses time-based electricity pricing.

Weather data / temperature sensors
Useful for predicting solar output and heating/cooling demand.

A practical way to think about it is in 4 groups:

Measure
Smart meter, submeters, current clamps, temperature sensors.

Generate
Solar inverter.

Store
Home battery, EV battery.

Control big loads
Heat pump, boiler, EV charger, dishwasher, washer, dryer, AC.

curl -X POST "http://192.168.1.13:4050/internal/jobs/sync"   -H "Authorization: Bearer 1234567890"   -H "Content-Type: application/json"   -d '{"jobs":[{"id":"test-solaredge-1","type":"solaredge_site","baseUrl":"http://192.168.1.22:3045","remoteSiteId":"1","bearerToken":"local-dev-key","instanceId":"default","pollIntervalMs":300000}]}'
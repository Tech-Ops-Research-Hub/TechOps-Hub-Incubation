# Technology Stacks & Architecture Playbook — Master Guide

Purpose: definitive, deployable stack recommendations for major product categories. Each section gives first-principles constraints, recommended stack (concrete components), architecture diagram (ASCII), sample development→production workflow, key implementation notes, scaling & operational patterns, security & compliance checklist, testing strategy, and minimal repo layout. Use this as a blueprint to start building immediately.

---

# How to read this guide

* Follow constraints → choose the stack section matching the product domain.
* Use the architecture diagram as canonical; adapt services to cloud vendor choices.
* Prioritize invariants: data integrity, availability, confidentiality, and auditability.

---

# 1. FinTech

## Fundamental constraints (first principles)

* Money = legal, auditable state. Invariants: no silent failures, full traceability, atomicity of critical transactions, non-repudiation, strong identity. Throughput and latency matter; regulatory constraints and KYC/AML apply. Fail-safe and rollback logic required.

## Recommended stack (concrete)

* **Backend**: Java + Spring Boot (primary), or Node.js with NestJS for rapid APIs; Go for high-throughput microservices.
* **Database**: PostgreSQL (primary, ACID), with write-ahead logging; read replicas for reporting. Use TimescaleDB only if time-series analytics needed.
* **Ledger / Event store**: Kafka or event-sourcing with an append-only store (Postgres append-only table or EventStoreDB).
* **Cache / Fast data**: Redis (clusters) for ephemeral state and rate-limiting.
* **Payments / PCI**: Stripe Connect / Adyen / local PSPs. Never store raw card data; use tokenization.
* **Auth & Identity**: OAuth2/OIDC provider, JWT for services, hardware-backed HSM for signing keys (or cloud KMS). Multi-factor auth (2FA) mandatory.
* **Infra**: Kubernetes + StatefulSets for DB, Vault for secrets, Terraform for infra as code.
* **Observability**: Prometheus, Grafana, ELK (or Loki), distributed tracing (OpenTelemetry + Jaeger).
* **Queueing / Tasks**: Kafka for high-throughput messaging; RabbitMQ or BullMQ for job queues.
* **Language/Libs for ML**: Python microservices for scoring (fraud/Risk) behind RPC/HTTP.

## Minimal architecture (ASCII)

```
[Clients] -> [API Gateway / LB] -> [Auth Service]
                                  -> [API Services (payments, ledger, accounts)]
                                       -> [Postgres (master + replicas)]
                                       -> [Kafka] -> [Workers]
                                       -> [Redis]
Workers -> [External PSPs / KMS / HSM]
Monitoring -> Prometheus/Grafana, Tracing -> Jaeger
```

## Sample dev→prod workflow

1. Define API contracts (OpenAPI) and event schemas (Avro/JSON Schema).
2. Implement services in feature branches. Unit tests + contract tests.
3. CI: lint → unit tests → build container → run integration tests against ephemeral DB (Testcontainers).
4. Deploy to staging with feature flags. Run load tests (k6).
5. Security audit, penetration test, compliance checklist. Approve deploy to production with DB migration plan.

## Key implementation notes

* Use DB transactions for account balance updates. Enforce row-level locks where needed.
* Use idempotency keys for external payment operations.
* Keep reconciliation job to compare ledger vs external provider daily.
* Use KYC microservice to store verified identity hashes, not raw documents.

## Scaling & performance

* Partition accounts by shard key when scale requires horizontal DB splits.
* Use CQRS for read-heavy dashboards: event-sourced writes, read replicas and materialized views for reporting.

## Security & compliance checklist

* PCI-DSS scope reduction: use PSP-hosted checkout/tokenization.
* TLS everywhere, strict CSP, HSTS.
* Key rotation via KMS/HSM.
* Audit logs immutable and exportable.
* Data residency and retention policies enforced.

## Testing

* Unit → Contract → Integration (Testcontainers) → E2E on staging → Load testing (k6).
* Chaos tests for failover.

## Repo layout (minimal)

```
/services
  /accounts-service
  /payments-service
  /fraud-service
/shared
  /schemas (OpenAPI, Avro)
infrastructure
  /terraform
k8s
ci/
```

---

# 2. AgriTech

## Fundamental constraints

* Heavy IoT and geospatial data. Invariants: sensor data correctness, temporal ordering, offline device behavior, network reliability in rural environments, cost sensitivity.

## Recommended stack

* **Backend**: Node.js (NestJS) for APIs and device protocol adapters; Python (FastAPI) for ML models/predictions.
* **Database**: PostgreSQL + PostGIS for spatial; Time-series: TimescaleDB or InfluxDB for sensor data.
* **Messaging / Telemetry**: MQTT broker (Eclipse Mosquitto or EMQX) for device publish-subscribe; Kafka for telemetry ingestion.
* **Edge / Device**: Lightweight agents (C/C++, Micropython) using MQTT + TLS; local caching on device; periodic sync.
* **Mobile apps**: React Native for farmer apps (offline sync).
* **Mapping/Visualization**: Mapbox or Leaflet with PostGIS queries.
* **Infra**: Kubernetes for cloud components; serverless functions for bursty processing.
* **Analytics**: Python notebooks, JupyterHub, ML pipelines (Airflow or Prefect).

## Architecture

```
[IoT Devices] --MQTT--> [MQTT Broker] -> [Ingest service] -> [Kafka]
                                           -> [Time-series DB]
                                           -> [Postgres/PostGIS]
[Web/Mobile] -> [API Gateway] -> [NestJS APIs] -> DBs
ML Models -> batch jobs in Airflow/Prefect -> feature store
```

## Sample workflow

* Devices publish telemetry → broker → ingestion service validates, stamps timestamps, writes to TSDB and events to Kafka.
* Batch jobs compute aggregated metrics and predictions stored in Postgres features.
* Mobile apps call APIs and sync cached state when connectivity resumes.

## Key implementation notes

* Always attach device metadata and sequence numbers to avoid replay or out-of-order issues.
* Implement backpressure on ingestion; use partitions per farm/region in Kafka.
* Design low-power communication and small payload formats (CBOR).

## Scaling & ops

* Use partitioned topics per region in Kafka.
* Use read replicas for heavy analytical queries.
* Plan data retention for raw telemetry vs aggregated summaries.

## Security & privacy

* Device auth via X.509 certificates or JWTs with short TTL.
* Secure OTA updates via signed firmware.
* Enforce role-based access for farmer/cooperative datasets.

## Testing

* Device simulators for local testing.
* Integration tests with mock MQTT broker and TSDB.

## Repo layout

```
/ingest
/api
/ml
/device-sdk
/infrastructure
```

---

# 3. HealthTech

## Fundamental constraints

* Patient data privacy (PHI), regulatory compliance (HIPAA, GDPR), interoperability (FHIR), high reliability. Data must be auditable and access-controlled.

## Recommended stack

* **Backend**: Python (Django/DRF or FastAPI) or Java (Spring Boot) depending on enterprise needs.
* **Database**: PostgreSQL; consider document store (MongoDB) for unstructured notes. Use FHIR store (HAPI FHIR or equivalent).
* **Interoperability**: FHIR APIs, HL7 connectors, OAuth2 for API access.
* **Auth**: OAuth2 + OpenID Connect, fine-grained RBAC, attribute-based access control (ABAC) patterns.
* **Messaging**: Kafka for integration and audit trail.
* **Storage**: Encrypted object storage for DICOM and imaging (S3-compatible). Use PACS systems for imaging integrations.
* **Infra**: Private VPC, strict network segmentation, Vault for secrets.

## Architecture

```
[Clients: clinicians/patients] -> [API Gateway] -> [Auth] -> [FHIR Service] -> [Postgres]
                                         -> [Imaging Service] -> [PACS / S3]
                                         -> [Audit Service] -> [Kafka]
Monitoring & logs must be isolated and protected.
```

## Workflow

* Clinician requests patient data → API enforces scope and consent → fetches from FHIR store and imaging service → logs all accesses to immutable audit log.

## Key implementation notes

* Encrypt data at rest and in transit.
* Minimal data exposure: return de-identified data where possible.
* Maintain consent records and data access purpose logs.
* Implement strong retention and deletion workflows.

## Compliance checklist

* Business Associate Agreements (BAA) with cloud providers where needed.
* Data breach response plan and logging with immutable storage.
* Periodic audits and penetration testing.

## Testing

* Contract tests against FHIR resource schemas.
* E2E tests that validate RBAC and consent flows.
* Mock external health systems for integration tests.

## Repo layout

```
/fhir-service
/auth
/imaging
/audit
/infrastructure
```

---

# 4. PropTech (Real Estate)

## Fundamental constraints

* Geospatial queries, SEO for listings, large media (images & tours), frequent reads (search), moderate write volume.

## Recommended stack

* **Backend**: Node.js (NestJS) or Python (Django) for content management.
* **Frontend**: Next.js for SSR/SSG to maximize SEO and performance.
* **Database**: PostgreSQL + PostGIS for geospatial; Elasticsearch for search and facets.
* **Media**: S3 + CDN; image optimization service (Thumbor or cloud provider image service). Use prerendered thumbnails.
* **Realtime**: WebSockets or server-sent events for messaging and bookings.
* **3D/VR**: Three.js / WebXR for tours; store assets in optimized tiling (glTF / Draco).

## Architecture

```
[Users] -> [Next.js] -> [API Gateway] -> [Listing Service] -> [Postgres (+PostGIS)]
                                                   -> [Elasticsearch]
                                                   -> [S3 / CDN]
                                                   -> [Booking & Messaging Service]
```

## Workflow

* Listing created by agent → image upload presigned → thumbnails generated and stored → listing indexed to Elasticsearch → available to front-end via SSR.

## Key implementation notes

* Use geohashes for efficient spatial queries combined with PostGIS.
* Keep image sizes optimized and use responsive image srcsets.
* Use server-side rendering for listing pages; cache aggressively with CDN and short invalidation on updates.

## Scaling & ops

* Read replicas + Elasticsearch cluster for search.
* Use autoscaling for front-end and booking services.

## Security & compliance

* Secure PII (owner contact) and ensure opt-out mechanisms.
* Verify listing ownership to combat fraud.

## Testing

* Search relevance tests, image pipeline integration tests, pagination and SEO meta test.

## Repo layout

```
/frontend (Next.js)
 /components
/backend
 /listings
 /media-worker
/search
/infrastructure
```

---

# 5. CleanTech (Energy & Sustainability)

## Fundamental constraints

* High-frequency sensor streams, real-time analytics, predictive models, offline reliability for edge devices.

## Recommended stack

* **Backend**: Go for high-throughput ingest; Python/RT for ML.
* **Time-series DB**: InfluxDB or TimescaleDB.
* **Streaming**: Kafka for ingest and processing. MQTT for device telemetry.
* **Edge/Device**: Lightweight edge gateways with local buffering.
* **Visualization**: React dashboards, Grafana for time-series visualizations.

## Architecture

```
[Devices] -> MQTT -> Ingest -> Kafka -> Stream processors -> TSDB
                                          -> Alerts & Actuators -> Control systems
[UI] -> API -> TSDB / Aggregates
```

## Workflow

* Devices stream to MQTT; ingest validates/normalizes → publishes to Kafka → real-time processors compute rolling aggregates and anomalies → push to alerting and dashboard.

## Key implementation notes

* Implement TTL and downsampling for raw telemetry.
* Use bounded queues to protect storage from bursts.
* Design control loops with safety limits and manual overrides.

## Scaling & ops

* Partition Kafka topics by region/site.
* Use autoscaling for stream processors and forecast services.

## Security

* Mutual TLS for device connections. Secure firmware updates.

## Testing

* Simulate device streams; validate alerting and anomalies.

## Repo layout

```
/ingest
/stream-processors
/dashboard
/ml
/infrastructure
```

---

# 6. EntertainmentTech (Streaming, Games)

## Fundamental constraints

* Low-latency, high-concurrency, real-time communication, heavy media delivery, stateful game servers for multiplayer.

## Recommended stack

* **Realtime / Game servers**: Go or C++ for performance; Node.js for control plane; use UDP (ENet) or WebRTC for peer connections.
* **Media**: HLS/DASH for VOD, WebRTC for live low-latency. Cloud transcoding (MediaConvert) or ffmpeg pipelines.
* **Scalable storage**: S3 + CDN; object lifecycle policies for VOD.
* **Auth / Presence**: Redis for ephemeral state (presence, leaderboards).
* **DB**: Cassandra or DynamoDB for high-write needs (leaderboards), PostgreSQL for relational data.
* **Streaming infra**: SFU/MCU (Janus, Jitsi, mediasoup) for scalable WebRTC.

## Architecture

```
[Clients] -- WebRTC --> [SFU/MCU Cluster] -> CDN / Recording
[Clients] -> [API Layer] -> [Matchmaking Service] -> [Game Servers (stateful)]
                                     -> [Redis] (presence)
                                     -> [Cassandra/Dynamo] (events)
```

## Workflow

* Client connects to matchmaking → assigned to game server → game state syncs via UDP/WebSocket → periodic snapshots persisted to DB for reconnection.

## Key implementation notes

* Design deterministic tick loops for authoritative servers.
* Use snapshots + command logs for state reconciliation.
* Offload heavy compute (physics) to specialized servers or client-side where safe.

## Scaling & ops

* Autoscale game servers by active sessions. Use relocation and session affinity.
* Use regional edge to reduce latency.

## Security

* Cheat detection, rate-limiting, encrypted transports for control messages.

## Testing

* Simulate thousands of concurrent sessions. Use synthetic load for match-making and media flows.

## Repo layout

```
/api
/matchmaking
/game-servers
/sfu
/infrastructure
```

---

# Cross-cutting concerns (applies to all domains)

## Observability

* Metrics: Prometheus + Grafana.
* Traces: OpenTelemetry + Jaeger.
* Logs: Structured JSON to ELK or Loki.
* SLOs & Alerts: define latency/error SLOs per critical endpoint and instrument dashboards.

## CI/CD

* Pipelines: build → test → QA/staging deploy → canary → prod.
* Tools: GitHub Actions/GitLab CI/Jenkins. Use image tags with SHA.

## Infrastructure

* IaC: Terraform for cloud resources; Helm for Kubernetes deployments.
* Secrets: HashiCorp Vault or cloud KMS.
* Backups & DR: automated, tested restores, and RTO/RPO targets.

## Security baseline

* TLS everywhere, WAF on ingress, rate limiting, input validation, dependency scanning (Snyk/Dependabot), DLP for sensitive data.
* Least privilege IAM policies, key rotation, audit trails.

## Data governance

* Data classification, retention rules, subject access requests, automated deletions, encryption at rest and in transit.

## Testing pyramid

* Unit tests (fast) → Integration tests (DB, services) → Contract tests (OpenAPI/Kafka schemas) → E2E tests (Cypress) → Load & Chaos tests.

## Cost & procurement considerations

* Use managed services to reduce ops footprint unless latency/cost dictates self-host.
* Estimate storage and egress costs (media-heavy products costlier).
* Choose regions for data residency and latency.

---

# Implementation checklist (practical, actionable)

1. Define invariants and SLOs for primary flows (login, transaction, stream, ingestion).
2. Create OpenAPI + event schemas and contract tests.
3. Select primary language and a minimal tech stack (DB, cache, message bus).
4. Scaffold monorepo or multi-repo following the repo layout examples.
5. Implement CI/CD pipeline with automated tests and staging deploy.
6. Implement observability, audit logs, and secrets management before accepting production traffic.
7. Run load tests that simulate expected peak + safety margin.
8. Security review and compliance gap analysis.

---

# Quick comparison matrix (decision cheat-sheet)

* Need strict ACID + audit → **FinTech: Postgres + Java**.
* Heavy IoT & geospatial → **AgriTech: Postgres+PostGIS + MQTT + TimescaleDB**.
* PHI + interoperability → **HealthTech: FHIR + Django/Spring + encrypted storage**.
* SEO + media-driven listings → **PropTech: Next.js + Postgres/PostGIS + Elasticsearch**.
* Real-time low-latency + streaming → **Entertainment: WebRTC + SFU + Go/C++ game servers**.
* Time-series sensor analytics → **CleanTech: Influx/Timescale + Kafka + Go ingest**.

---

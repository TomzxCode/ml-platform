---
title: "API Server: Api"
status: draft
structure: split
concern: "api"
---

# Plan: Api

## Goal

Implement the server surface defined by the contract concern: a FastAPI application with token authentication, workspace scoping, structured request logging, streaming, and a gRPC log streaming service.
The server package lives alongside the CLI package in this repository (`src/<server package>/`, per conventions) and persists everything through the database concern.

## Phases

### Phase 1: Scaffold and middleware

**Goal:** A running FastAPI app with the cross-cutting machinery every endpoint inherits.
**Effort:** 3 person-days
**Depends on:** contract Phase 1 (error model, auth scheme)

**Deliverables:**
- [ ] FastAPI app factory, settings, and lifespan in a server package with src layout
- [ ] Request-id middleware: structured logs (structlog) with a request identifier echoed in every response (NFR-3)
- [ ] Token verification middleware against hashed-at-rest tokens; token values never logged (FR-2, NFR-1)
- [ ] Unified exception handling producing the contract error model

### Phase 2: Auth and workspaces

**Goal:** Authentication and workspace isolation enforced end to end.
**Effort:** 3 person-days
**Depends on:** Phase 1, database Phase 1

**Deliverables:**
- [ ] `POST /auth/token` issuance and `DELETE /auth/token` revocation with expiry (FR-2, FR-13, open question 2 resolution)
- [ ] `GET /users/me` and `GET /workspaces` endpoints (FR-2, FR-3)
- [ ] Workspace scoping enforced on every request; cross-workspace access denied (FR-3)

### Phase 3: Catalog endpoints

**Goal:** Every non-job resource operable through the API.
**Effort:** 4 person-days
**Depends on:** Phase 2, contract Phase 2, database Phase 2

**Deliverables:**
- [ ] Dataset catalog endpoints: create, list, get, delete (FR-3)
- [ ] Secret endpoints: set, list names, delete; values never returned or logged (FR-3, NFR-1)
- [ ] Compute type endpoints with GPUs, memory, and quota (FEAT-p1 FR-18)
- [ ] Experiment endpoints with runs and metrics (FEAT-p1 FR-19)
- [ ] Model registry endpoints: register by artifact reference, list, get versions with lineage (FR-7)

### Phase 4: Job endpoints

**Goal:** Submission and lifecycle operations for all three job types.
**Effort:** 3 person-days
**Depends on:** Phase 3, contract Phase 3, database Phase 3

**Deliverables:**
- [ ] `POST /jobs` for data processing, training, and batch inference specs, with validation errors naming the offending field (FR-4, FR-6, FR-8)
- [ ] Idempotency key handling on submission (open question 5 semantics)
- [ ] Job list with type and state filters, cursor pagination; job get (FR-10, FR-14)
- [ ] Job cancel transitioning to cancelled state (FR-10)

### Phase 5: Streaming and transfers

**Goal:** Log streaming and transfer operations.
**Effort:** 3 person-days
**Depends on:** Phase 4, contract Phase 3

**Deliverables:**
- [ ] SSE log streaming with heartbeat; gRPC `JobLogs` service over the same log store (FR-10)
- [ ] Transfer submission and status endpoints delegating execution to the dispatch concern (FR-5)
- [ ] Optional May: state-change event stream for jobs and deployments (FR-15)

### Phase 6: Hardening

**Goal:** NFR gates green and CI wired.
**Effort:** 2 person-days
**Depends on:** Phases 1 to 5

**Deliverables:**
- [ ] Submission acknowledgment under 1 second at p95 under load test (NFR-4)
- [ ] k8s deployment manifests for the stateless server pods, with a documented rollout and rollback procedure (NFR-2)
- [ ] Contract test suite green against the running server (NFR-5)
- [ ] GHA running `ruff check`, `ruff format --check`, `ty check`, `pytest` (NFR-5)

## Risk Register (concern-local)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Streaming endpoints block the event loop under concurrent followers | Med | Med | Async endpoints end to end; concurrent-follower load test in Phase 6 |
| Token format chosen in Phase 2 mismatches the hashed storage design | Low | Med | Decide the token format jointly with the database concern before Phase 2 lands |

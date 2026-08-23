---
title: "API Server: Contract"
status: draft
structure: split
concern: "contract"
---

# Plan: Contract

## Goal

Define the complete, versioned API contract for the platform: an OpenAPI 3 document for REST covering all five operation families, plus proto definitions for gRPC log streaming.
The contract is the source of truth shared by this server and FEAT-p1, whose command surface (p1 plan, Command reference) defines the first consumer.
Every FEAT-p1 command must map to a contract operation; this concern publishes the contract before any server implementation begins.

## API Surface: FEAT-p1 CLI Mapping

Workspace-scoped resources live under `/workspaces/{workspace}/...`; identifiers in paths are user-visible names.

| FEAT-p1 command | Contract operation | Maps to |
|---|---|---|
| auth login | `POST /auth/token` | FR-2, FR-13 |
| auth logout | `DELETE /auth/token` | FR-13 |
| auth status | `GET /users/me` | FR-2 |
| workspace list | `GET /workspaces` | FR-3 |
| workspace select, workspace get | `GET /workspaces/{workspace}` | FR-3 |
| dataset create | `POST /workspaces/{workspace}/datasets` | FR-3 |
| dataset list | `GET /workspaces/{workspace}/datasets` | FR-3, FR-14 |
| dataset get | `GET /workspaces/{workspace}/datasets/{dataset}` | FR-3 |
| dataset delete | `DELETE /workspaces/{workspace}/datasets/{dataset}` | FR-3 |
| data copy | `POST /workspaces/{workspace}/transfers`, `GET /transfers/{transfer}` | FR-5 |
| secret set | `POST /workspaces/{workspace}/secrets` | FR-3 |
| secret list | `GET /workspaces/{workspace}/secrets` (names only) | FR-3 |
| secret delete | `DELETE /workspaces/{workspace}/secrets/{secret}` | FR-3 |
| compute list | `GET /workspaces/{workspace}/compute-types` | FR-3, FR-11 |
| compute get | `GET /workspaces/{workspace}/compute-types/{compute_type}` | FR-3, FR-11 |
| job submit, batch submit | `POST /workspaces/{workspace}/jobs` (job type declared by the spec) | FR-4, FR-6, FR-8 |
| job list, batch list | `GET /workspaces/{workspace}/jobs?type=...&state=...` | FR-10, FR-14 |
| job get, batch get | `GET /jobs/{job}` | FR-10 |
| job logs, batch logs | `GET /jobs/{job}/logs` (SSE); gRPC `JobLogs.Stream` | FR-10 |
| job cancel, batch cancel | `POST /jobs/{job}/cancel` | FR-10 |
| experiment create | `POST /workspaces/{workspace}/experiments` | FR-3 |
| experiment list | `GET /workspaces/{workspace}/experiments` | FR-3, FR-14 |
| experiment get | `GET /workspaces/{workspace}/experiments/{experiment}` | FR-3 |
| deploy create | `POST /workspaces/{workspace}/deployments` | FR-9 |
| deploy list | `GET /workspaces/{workspace}/deployments` | FR-9, FR-14 |
| deploy get | `GET /workspaces/{workspace}/deployments/{deployment}` | FR-9 |
| deploy update | `PATCH /workspaces/{workspace}/deployments/{deployment}` | FR-9 |
| deploy stop | `POST /workspaces/{workspace}/deployments/{deployment}/stop` | FR-9 |
| model register | `POST /workspaces/{workspace}/models` (registered by artifact location reference) | FR-7 |
| model list | `GET /workspaces/{workspace}/models` | FR-7, FR-14 |
| model get | `GET /workspaces/{workspace}/models/{model}` | FR-7 |
| completion | none (client-side only) | |

## Phases

### Phase 1: Foundations

**Goal:** Contract skeleton with cross-cutting conventions settled and the two resolvable open questions decided.
**Effort:** 2 person-days
**Depends on:** None

**Deliverables:**
- [ ] OpenAPI 3 document skeleton: bearer-token security scheme, media types, versioning, resource naming (FR-1, FR-2)
- [ ] Error model: code, message, likely cause, suggested fix, echoed on every failure (FEAT-p1 NFR-1 alignment)
- [ ] Pagination convention: cursor-based, applied to every list operation (FR-14)
- [ ] Idempotency convention: `Idempotency-Key` header on submissions with replay semantics (open question 5)
- [ ] Decisions recorded for open questions 4 (REST first, gRPC for log streaming) and 5 via `/create-decision`

### Phase 2: Catalog resources

**Goal:** Schemas and paths for every non-job resource.
**Effort:** 3 person-days
**Depends on:** Phase 1

**Deliverables:**
- [ ] Workspace, dataset, secret, compute type, experiment, and run schemas, workspace-scoped (FR-3)
- [ ] Explicit dataset registration endpoints (per requirements review observation), with location, format, and version fields
- [ ] Secret schemas accept values on write and never return them on read (NFR-1 alignment)
- [ ] Model and model version schemas with artifact location and lineage to the producing job (FR-7)

### Phase 3: Jobs and transfers

**Goal:** The job lifecycle contract for all three job types plus data transfer.
**Effort:** 3 person-days
**Depends on:** Phase 2

**Deliverables:**
- [ ] Submission schemas for data processing, training, and batch inference jobs (code or model reference, dataset reference, compute selection) (FR-4, FR-6, FR-8)
- [ ] Job state model (queued, running, succeeded, failed, cancelled) with state, list, get, and cancel operations (FR-10)
- [ ] Log streaming contract: SSE endpoint schema with heartbeat and gRPC service sketch (FR-10)
- [ ] Transfer request and resume semantics matching FEAT-p1 FR-13 (FR-5)

### Phase 4: Deployments and gRPC publication

**Goal:** The serving contract, the finished protos, and a validated handoff to FEAT-p1.
**Effort:** 2 person-days
**Depends on:** Phase 3

**Deliverables:**
- [ ] Deployment and endpoint schemas: served model version, static replica count, autoscaling bounds (min/max), state (FR-9)
- [ ] gRPC protos for log streaming (and an optional job state-change event service, May) (FR-10, FR-15)
- [ ] Contract test suite skeleton runnable in CI (NFR-5)
- [ ] Contract review against the full FEAT-p1 command reference; stub generation validated (closes assumptions 1 and 2)

## Risk Register (concern-local)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Contract specifies behavior the server cannot deliver in v1 (resume, events) | Med | Med | Mark Should and May capabilities explicitly; the server may return a documented not-implemented error for May items |
| Artifact registration semantics (upload vs location reference) decided wrong | Low | Med | Location reference in v1; revisit after the storage backend decision (open question 1) |

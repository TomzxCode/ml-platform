---
title: "API Server: Database"
status: draft
structure: split
concern: "database"
---

# Plan: Database

## Goal

Provide the PostgreSQL persistence layer all durable state lives in: identity and workspaces, catalog resources, job and deployment state, with migrations, state-machine constraints, and restart durability.
The schema implements the domain entities from `architecture.md` and serves the api concern's endpoints; pods hold no durable state (NFR-2).

## Phases

### Phase 1: Baseline schema and migrations

**Goal:** Migration tooling in CI and the identity tables live.
**Effort:** 2 person-days
**Depends on:** None

**Deliverables:**
- [ ] Migration tooling (alembic) wired into the server package and CI
- [ ] Tables: teams, users, workspaces, workspace memberships (FR-3)
- [ ] Tokens table storing hashes only, with expiry and revocation columns (FR-2, FR-13, NFR-1)

### Phase 2: Catalog tables

**Goal:** Persistence for every catalog resource.
**Effort:** 3 person-days
**Depends on:** Phase 1, contract Phase 2

**Deliverables:**
- [ ] Tables: datasets, secrets (encrypted at rest), compute types, workspace quotas (FR-3)
- [ ] Tables: experiments, runs, run metrics (FR-3)
- [ ] Tables: models, model versions with artifact location and lineage to the producing job (FR-6, FR-7)

### Phase 3: Job and transfer state

**Goal:** Durable job lifecycle with constraints.
**Effort:** 3 person-days
**Depends on:** Phase 2, contract Phase 3

**Deliverables:**
- [ ] Jobs table covering data processing, training, and batch inference (single typed table or per-type, decided here) (FR-4, FR-6, FR-8)
- [ ] Job events and job logs tables backing state reporting and streaming (FR-10)
- [ ] Transfers table with progress and resume checkpoints (FR-5)
- [ ] State machine constraints on job state transitions; unique idempotency keys per workspace (FR-10)

### Phase 4: Deployment state

**Goal:** Persistence for online inference.
**Effort:** 1 person-day
**Depends on:** Phase 2

**Deliverables:**
- [ ] Deployments and endpoints tables with served model version, replica count, and autoscaling bounds (FR-9)

### Phase 5: Durability

**Goal:** Restart survival proven and retention under control.
**Effort:** 2 person-days
**Depends on:** Phase 3

**Deliverables:**
- [ ] Restart survival test: pod kill leaves all job state unchanged (FR-12, NFR-2)
- [ ] Outbox table feeding the optional event stream (FR-15, May)
- [ ] Job log retention policy and purge job
- [ ] Backup and restore procedure documented

## Risk Register (concern-local)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Job log volume grows unbounded in PostgreSQL | Med | Med | Retention policy in Phase 5; evaluate object storage for logs when volume grows |
| Schema churn while the contract stabilizes forces repeated migrations | Med | Low | Every change lands as a migration; contract is the source of truth for resource fields |

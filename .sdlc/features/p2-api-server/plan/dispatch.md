---
title: "API Server: Dispatch"
status: draft
structure: split
concern: "dispatch"
---

# Plan: Dispatch

## Goal

Execute the work the api concern accepts: a scheduler that makes placement decisions and dispatches jobs to Kubernetes, training artifact handling with model registration, batch inference execution, data transfer execution, and online inference serving.
The compute backend and artifact storage sit behind interfaces so the undecided cloud provider (open question 3) and storage backend (open question 1) do not block progress.

## Phases

### Phase 1: Backend abstraction and fake

**Goal:** Interfaces plus a fake implementation that unblocks all later phases without real infrastructure.
**Effort:** 2 person-days
**Depends on:** None

**Deliverables:**
- [ ] `ComputeBackend` interface: submit, status, cancel; in-process fake for dev and tests (FR-11, open question 3 deferral)
- [ ] `ArtifactStorage` interface with an S3-compatible implementation (MinIO for dev) (open question 1 deferral)

### Phase 2: Scheduler and Kubernetes dispatch

**Goal:** Jobs placed and running on k8s, observable, and reconciled after restarts.
**Effort:** 5 person-days
**Depends on:** Phase 1, api Phase 4, database Phase 3

**Deliverables:**
- [ ] Submission queue and placement decision against workspace compute quota (FR-11)
- [ ] k8s Job dispatch with resource requests derived from the compute type (FR-11)
- [ ] Reconciliation loop matching k8s state to database state on startup (NFR-2)
- [ ] Log collection from dispatched jobs feeding the log store behind streaming (FR-10)
- [ ] kind-based integration test in CI

### Phase 3: Training artifacts

**Goal:** Training jobs produce registered models.
**Effort:** 3 person-days
**Depends on:** Phase 2

**Deliverables:**
- [ ] Checkpoint tracking to artifact storage (FR-6)
- [ ] Final checkpoint registered as a model version with lineage to the training job (FR-6, FR-7)
- [ ] Experiment and run metrics recorded during training (FEAT-p1 FR-19)

### Phase 4: Batch inference and transfers

**Goal:** The remaining two job families execute end to end.
**Effort:** 3 person-days
**Depends on:** Phase 2

**Deliverables:**
- [ ] Batch inference execution over a registered model version and dataset (FR-8)
- [ ] Transfer execution with progress reporting and resume from checkpoints (FR-5, FEAT-p1 FR-13)

### Phase 5: Online inference

**Goal:** Deployments serve traffic and are manageable.
**Effort:** 4 person-days
**Depends on:** Phase 3

**Deliverables:**
- [ ] Deployment controller creating a k8s serving deployment per model version, with endpoint provisioning (FR-9)
- [ ] Static replica count and autoscaling bounds mapped to k8s scaling (FEAT-p1 FR-10)
- [ ] Update served model version without dropping the endpoint; stop releases resources (FR-9)

## Risk Register (concern-local)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Kubernetes API complexity and RBAC scope slow Phase 2 | Med | High | Narrow RBAC profile; kind integration tests from the first k8s dispatch |
| Autoscaling semantics differ between contract fields and k8s behavior | Med | Med | Spike in Phase 5; keep contract fields to min/max replicas |
| Cloud provider decision (open question 3) lands late | High | Med | Backend interface keeps v1 k8s-only; a new backend is additive |

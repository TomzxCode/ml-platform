---
status: Active
---

# Assumption: Kubernetes is the only dispatch target required in v1

**Date:** 2026-08-23
**Status:** Active
**Author:** ml-platform implementors

---

## Statement

v1 dispatch targets a single Kubernetes cluster only; no cloud-provider-specific APIs are required, and the cloud provider decision (FEAT-p2 open question 3) can stay deferred behind the `ComputeBackend` interface without rework.

## Basis

The FEAT-p2 requirements name Kubernetes deployment as a constraint.
The requirements review flags FR-11 (placement and dispatch) as the largest implementation risk and bounds it with open questions 3 and 4.
Kubernetes Jobs plus standard scaling cover all three job families and online serving.

## Confidence

**Level:** Medium

Kubernetes sufficiency is technically well supported, but the cloud provider decision is organizational and still open.

## Risk if Wrong

**Impact:** Medium

If a cloud API (managed batch or managed serving) is mandated for v1, the dispatch concern reworks its placement and serving phases against that API; the backend interface limits the rework to new backend implementations.

## Validation Plan

**Method:** Resolve open question 3 with a decision record during specifications; confirm the chosen provider's managed services are not mandatory for v1.
**Target Date:** 2026-09-06

## Outcome

---
title: "mlx batch command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx batch`

## Overview

The `batch` group is the batch-inference fast path: submit a batch inference spec and mirror the job lifecycle commands for batch jobs.
It implements FEAT-p1 FR-8, FR-11, and FR-14.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `batch submit` | `<spec-file>` | Submit a batch inference job; returns a job id |
| `batch list` | `[--state <state>]` | List batch inference jobs |
| `batch get` | `<id>` | Print one batch job's current state |
| `batch logs` | `<id> [--follow]` | Stream a batch job's logs |
| `batch cancel` | `<id>` | Cancel a batch job |

## Spec submitted to `batch submit`

The same YAML document as the job spec with `type: batch-inference`, defined normatively in FEAT-p9 (`mlx job`).
The schema is shared, not forked: FEAT-p9 owns it, and this feature references it so the two submission paths never drift.

Fields recap (authoritative table in FEAT-p9):

| Field | Required | Constraints |
|---|---|---|
| model | Yes | `name:version` of a registered model (FR-7) |
| inputs[].dataset | Yes | Dataset catalog name scored by the model |
| output.dataset | Yes | Output dataset name registered on success |
| output.location | No | Explicit destination URI |
| Common fields (name, compute, secrets, env) | As in FEAT-p9 | |

Example:

```yaml
name: nightly-ctr-scoring
type: batch-inference
compute:
  type: gpu-a100-40g
model: ctr-estimator:3
inputs:
  - dataset: cleaned-clicks
output:
  dataset: nightly-scores
```

## Local validation

Identical to `job submit` validation in FEAT-p9, restricted to the batch-inference branch: local schema and enum checks first, then server-side confirmation of the model and dataset references.

## Equivalence

`batch list` output equals `job list --type batch-inference`, and `batch get/logs/cancel` accept the same job ids as their `job` counterparts (FEAT-p1 plan, command reference).

## Open questions

- None specific to this group beyond those listed in FEAT-p9.

---
title: "mlx job command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx job`

## Overview

The `job` group submits jobs defined by a user-supplied spec covering data processing, training, and batch inference, and reports their state (list, inspect, stream logs, cancel).
It implements FEAT-p1 FR-4, FR-6, FR-8, FR-11, and FR-14.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `job submit` | `<spec-file>` | Submit a job; returns a job id |
| `job list` | `[--type <type>] [--state <state>]` | List jobs with id, type, state |
| `job get` | `<id>` | Print one job's current state |
| `job logs` | `<id> [--follow]` | Print logs; `--follow` keeps the stream open |
| `job cancel` | `<id>` | Request cancellation |

## Spec submitted to `job submit`

One YAML file, discriminated by the `type` field: `processing`, `training`, or `batch-inference`.

### Common fields (all types)

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case, shown in listings |
| type | string | Yes | `processing`, `training`, or `batch-inference` |
| compute.type | string | Yes | A type from `mlx compute list` (FR-18) |
| compute.count | integer | No | Defaults to 1 |
| description | string | No | Free-form |
| secrets | string list | No | Names from `mlx secret list`; values never appear (FR-17) |
| env | string map | No | Environment variables for the job |

### `type: processing` fields (FR-4)

| Field | Type | Required | Constraints |
|---|---|---|---|
| inputs[].dataset | string | Yes | Dataset catalog name |
| inputs[].mount | string | No | Path the job reads the input from |
| outputs[].name | string | Yes | Dataset name registered in the catalog on success |
| outputs[].location | string | No | Explicit destination URI; generated when omitted |
| code.image | string | Yes | Container image to run |
| code.entrypoint | string | No | Overrides the image entrypoint |
| code.args | string list | No | Arguments passed to the entrypoint |

### `type: training` fields (FR-6)

| Field | Type | Required | Constraints |
|---|---|---|---|
| code.image | string | Yes | Container image to run |
| code.entrypoint | string | No | Overrides the image entrypoint |
| code.args | string list | No | Arguments passed to the entrypoint |
| dataset | string | Yes | Dataset catalog name trained on |
| experiment | string | No | Experiment name the run attaches to (FR-19) |
| model.name | string | No | Registers the trained artifact under this name (FR-7) |
| hyperparameters | free-form map | No | Passed through to the training code |

### `type: batch-inference` fields (FR-8)

| Field | Type | Required | Constraints |
|---|---|---|---|
| model | string | Yes | `name:version` of a registered model (FR-7) |
| inputs[].dataset | string | Yes | Dataset catalog name scored by the model |
| output.dataset | string | Yes | Output dataset name registered on success |
| output.location | string | No | Explicit destination URI |

### Example (training)

```yaml
name: ctr-baseline-run
type: training
compute:
  type: gpu-a100-40g
  count: 2
code:
  image: ghcr.io/acme/ctr-trainer:1.4
  entrypoint: python train.py
dataset: cleaned-clicks
experiment: ctr-baseline
model:
  name: ctr-estimator
hyperparameters:
  lr: 0.001
  epochs: 10
secrets: [wandb-key]
```

## Local validation

Runs before any server call; each failure exits 1 and names the offending field (NFR-1):

- The file parses as YAML.
- Required fields per type are present, including per-type fields selected by `type`.
- Enums are valid (`type`, `compute.type` syntax).
- References (dataset names, model `name:version` syntax, secret names, compute type) are structurally valid locally, then confirmed against the server, which owns the authoritative check; unknown references are named in the error (FR-4, FR-6 scenarios).

## Open questions

- Whether `code` also accepts a source repository reference in addition to a container image.
- Mount path semantics per input (server contract, FEAT-p2).
- Whether `hyperparameters` is constrained (string and number values only).

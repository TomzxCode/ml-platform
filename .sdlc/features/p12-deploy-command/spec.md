---
title: "mlx deploy command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx deploy`

## Overview

The `deploy` group manages online inference deployments: create an endpoint for a registered model, inspect, update the served model or scaling, and stop.
It implements FEAT-p1 FR-9 and FR-10.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `deploy create` | `<model>:<version> [--endpoint-name <name>] [--replicas <n>] [--min-replicas <n>] [--max-replicas <n>]` | Deploy for online inference; prints the endpoint reference |
| `deploy list` | | List deployments with state |
| `deploy get` | `<name>` | Inspect one deployment |
| `deploy update` | `<name> [--model <model>:<version>] [--replicas <n>] [--min-replicas <n>] [--max-replicas <n>]` | Update served model and/or scaling |
| `deploy stop` | `<name>` | Stop the deployment |

No spec file is submitted to this group; the submitted input is the model reference and scaling flags.

## Input contract

### Model reference

| Input | Rules |
|---|---|
| `<model>:<version>` | Kebab-case model name plus version; must exist in the registry (FR-7) |

### Scaling flags (create and update)

| Flag | Rules |
|---|---|
| `--replicas <n>` | Static replica count; mutually exclusive with `--min-replicas`/`--max-replicas` |
| `--min-replicas <n>` | Autoscale lower bound; must be less than or equal to max |
| `--max-replicas <n>` | Autoscale upper bound |
| `--endpoint-name <name>` | Kebab-case; generated when omitted |

Setting `--replicas` clears autoscaling; setting bounds replaces a static count (FR-10 scenarios).

## Local validation

- The model reference parses as `name:version`; exit 1 with the expected form otherwise.
- `--replicas` together with either bound flag is a usage error.
- min greater than max is rejected locally with a clear error (FR-10).
- Existence of the model and deployment name is confirmed server-side.

## Update semantics

`deploy update` accepts any subset of the flags; omitted flags keep their current values.
Updating the served model triggers a rollout whose progress is visible via `deploy get`.

## Open questions

- Whether rollout progress streams during `deploy update` or is only visible via `deploy get`.
- Whether `deploy stop` requires confirmation or a `--yes` flag.

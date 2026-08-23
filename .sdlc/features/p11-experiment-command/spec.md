---
title: "mlx experiment command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx experiment`

## Overview

The `experiment` group creates experiments, lists them, and inspects one experiment's runs and their metrics.
It implements FEAT-p1 FR-19.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `experiment create` | `<spec-file>` | Create an experiment in the workspace |
| `experiment list` | | List experiments in the workspace |
| `experiment get` | `<name>` | Print one experiment's runs and their metrics |

## Spec submitted to `experiment create`

One YAML file describing the experiment.

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case, unique in the workspace |
| description | string | No | One line shown in listings |

Example:

```yaml
name: ctr-baseline
description: First pass on CTR estimation
```

## Local validation

- The file parses as YAML; exit 1 naming the parse error otherwise.
- `name` is present and kebab-case.
- Uniqueness is confirmed server-side on create.

## Relationship to training jobs

Training runs attach to an experiment by name via the training spec's `experiment` field (FEAT-p9).
An experiment name referenced by a training spec but never created is an open design question (attach-time auto-create versus error).

## Open questions

- Whether metrics come from the server's run records only, or the CLI can also parse run logs.
- Auto-create behavior for unknown experiment names in training specs.

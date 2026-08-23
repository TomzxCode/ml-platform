---
title: "mlx compute command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx compute`

## Overview

The `compute` group discovers compute types available to the workspace and inspects their resources and quota.
It implements FEAT-p1 FR-18.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `compute list` | | List machine types available to the workspace |
| `compute get` | `<type>` | Print one machine type's details (GPUs, memory, quota) |

No spec file is submitted to this group; the submitted input is the compute type name.

## Input contract

| Input | Applies to | Required | Rules |
|---|---|---|---|
| Type name | `get` | Yes | Kebab-case string, must match an entry from `compute list` |

The authoritative set of type names comes from the server, so unknown names fail server-side with the closest match suggested.

## Output

- `list`: a table of type, GPU model and count, memory, and remaining quota.
- `get`: a key-value block for one type, plus its quota.
- `--json`: an array of type objects, or a single object for `get` (FR-12).

## Open questions

- Whether `list` supports filters (for example GPU-only) in v1 or defers them.

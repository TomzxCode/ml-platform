---
title: "mlx model command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx model`

## Overview

The `model` group registers externally trained model artifacts in the registry and lists and inspects registered models.
It implements FEAT-p1 FR-7.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `model register` | `<artifact-path> --name <name> [--version <v>]` | Register an externally trained artifact |
| `model list` | | List registered models |
| `model get` | `<name>` | Print versions and artifact locations |

No spec file is submitted to this group; the submitted input is the artifact path, name, and optional version.

## Input contract

### `model register`

| Input | Required | Rules |
|---|---|---|
| artifact path | Yes | Local path or storage URI, accessible to the CLI |
| `--name` | Yes | Registry name, kebab-case, reused across versions |
| `--version` | No | Defaults to the next version under the name |

### `model list` and `model get`

| Input | Required | Rules |
|---|---|---|
| name | `get` only | Must match a registered model |

## Local validation

- The artifact path exists locally or uses a recognized URI scheme.
- `--name` is kebab-case.
- An explicit `--version` that already exists is rejected server-side as a duplicate.

## Output

- `list`: a table of model names and version counts.
- `get`: one block per version with its artifact location and creation time (FR-7 scenario).
- `--json`: an array of model objects, or a single object with a versions array for `get` (FR-12).

## Open questions

- Version scheme (integer sequence versus semver) and its default bump rule.
- Whether registering from a local path uploads the artifact or records the path only.

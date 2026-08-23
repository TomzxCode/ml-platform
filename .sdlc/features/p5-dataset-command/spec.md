---
title: "mlx dataset command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx dataset`

## Overview

The `dataset` group manages dataset catalog entries: create from a spec, list, inspect, delete.
It implements FEAT-p1 FR-20.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `dataset create` | `<spec-file>` | Register a dataset in the catalog from a spec |
| `dataset list` | | List datasets in the workspace |
| `dataset get` | `<name>` | Print one dataset's details (location, format, version) |
| `dataset delete` | `<name>` | Remove a dataset from the catalog |

## Spec submitted to `dataset create`

One YAML file describing the catalog entry.

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case, unique in the workspace |
| location | string | Yes | Storage URI, for example `s3://`, `gs://`, or `file://` |
| format | string | Yes | One of the catalog's known formats (parquet, csv, jsonl) |
| description | string | No | One line shown in listings |
| labels | string list | No | Free-form tags for filtering |

Example:

```yaml
name: raw-clicks
location: s3://ml-bucket/raw/clicks
format: parquet
description: Raw click events from the collector
labels: [daily, clicks]
```

## Local validation

Runs before any server call (FEAT-p1 plan, Phase 3):

- The file parses as YAML; exit 1 naming the parse error otherwise.
- Every required field is present; the error names the first missing field (NFR-1).
- `format` is a known format; the error lists the accepted values.
- `location` uses a recognized URI scheme.
- `name` is kebab-case and unique; uniqueness is confirmed server-side.

## Deletion semantics

`delete` removes the catalog entry only, never the underlying data.
Jobs later referencing the deleted name fail with an error naming the missing dataset (FR-20).

## Open questions

- Whether `delete` requires confirmation or a `--yes` flag.
- Whether the catalog versions entries, so re-creating the same name bumps a version instead of erroring.

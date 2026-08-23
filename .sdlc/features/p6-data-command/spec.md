---
title: "mlx data command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx data`

## Overview

The `data` group copies a dataset from a source location to a destination location, with progress and resumption.
It implements FEAT-p1 FR-5 and FR-13.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `data copy` | `<source> <destination> [--resume]` | Copy data between two locations |

No spec file is submitted to this group; the submitted input is the location pair plus flags.

## Input contract

| Input | Required | Rules |
|---|---|---|
| source | Yes | Storage URI readable by the user |
| destination | Yes | Storage URI writable by the user |
| `--resume` | No | Continue an interrupted copy instead of restarting |

## Local validation

- Both arguments parse as URIs with recognized schemes; exit 1 otherwise.
- Scheme pair must be supported (any-to-any is the goal; unrecognized pairs are rejected before any transfer).
- Read permission on the source and write permission on the destination are checked before transfer starts (FR-5).
- `--resume` with no recorded state for this source-destination pair behaves like a fresh copy, with one informational line.

## Behavior

- Progress (bytes copied, rate) goes to stderr in human mode and never to stdout (FR-13).
- On success, human mode prints a summary; `--json` prints one JSON summary document (FR-12).
- An interrupted copy leaves state sufficient to resume; the state is keyed by source and destination.

## Open questions

- The exact resume protocol (chunk ledger kept locally versus server-supported range resume); FEAT-p1 risk register flags backend support as a risk.
- Maximum concurrent transfers per copy.

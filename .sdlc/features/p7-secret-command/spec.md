---
title: "mlx secret command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx secret`

## Overview

The `secret` group manages workspace secrets that jobs reference for credentials, without ever displaying secret values.
It implements FEAT-p1 FR-17 and NFR-3.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `secret set` | `<name> (--from-literal <value> \| --from-file <path>)` | Store a credential in the workspace |
| `secret list` | | List secret names only, never values |
| `secret delete` | `<name>` | Remove a secret from the workspace |

No spec file is submitted to this group; the submitted input is the name plus exactly one value source.

## Input contract

### `secret set`

| Input | Required | Rules |
|---|---|---|
| name | Yes | Kebab-case, unique in the workspace |
| `--from-literal <value>` | Exactly one of the two sources | Value supplied on the command line; documented as a shell-history risk |
| `--from-file <path>` | | Value is the file's bytes; the file must exist and be readable |

### `secret list` and `secret delete`

| Input | Required | Rules |
|---|---|---|
| name | `delete` only | Must match an existing secret |

## Local validation

- Exactly one source flag on `set`; both or neither is a usage error (exit 1).
- The `--from-file` path exists and is readable.
- The name is kebab-case.
- Values never appear in any output, log, or error, including verbose mode (NFR-3).

## Deletion semantics

`delete` removes the secret immediately.
Jobs referencing the deleted name fail with a clear error naming the missing secret (FR-17).

## Open questions

- Whether to add an interactive masked prompt as a third source, to keep values out of shell history entirely.

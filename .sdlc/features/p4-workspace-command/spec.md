---
title: "mlx workspace command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx workspace`

## Overview

The `workspace` group selects the workspace that scopes every subsequent operation.
It implements FEAT-p1 FR-3.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `workspace list` | | List workspaces the user can access |
| `workspace select <name>` | `<name>` | Persist the active workspace in the active profile |
| `workspace get` | | Print the active workspace |

No spec file is submitted to this group; the submitted input is the workspace name.

## Input contract

| Input | Applies to | Required | Rules |
|---|---|---|---|
| Workspace name | `select` | Yes | Kebab-case string; must match an entry from `workspace list` |

The selection is stored in the active profile so later commands inherit it (FR-3).
`list` and `get` take no command-specific input.

## Local validation

- No workspace selected when an operation command runs: exit 1 with the fix naming `mlx workspace select`.
- Unknown name on `select`: exit 3 after the server rejects it, with the closest matching name suggested.

## Output

- `list`: a table of name and description.
- `get`: the active workspace name, or a clear statement that none is selected.
- `--json`: an array of workspace objects, or a single object for `get` (FR-12).

## Open questions

- Whether a default workspace is auto-selected when the user belongs to exactly one.

---
title: "mlx experiment command"
status: draft
parent: FEAT-p1
---

# CLI Design: mlx experiment command

## Design Principles

- Extends the `mlx` CLI conventions fixed by the FEAT-p1 plan: kebab-case command names, verb-first subcommands under a noun group, GNU-style long options, cyclopts as the framework.
- Exit codes follow the CLI-wide scheme (0 success, 1 usage error, 2 authentication failure, 3 server error, 4 cancelled by user).
- Human output on stdout, diagnostics on stderr; one valid JSON document on stdout when `--json` is passed and nothing else.
- The experiment roster and run records are authoritative on the server; the CLI validates only locally checkable rules (YAML parse, name presence, kebab-case shape) before calling the server.
- The group is read-only toward runs: it renders what the server recorded, never parses run logs or recomputes metrics (default for open question 1).

## Command Tree

```
mlx experiment
├── create    Create an experiment in the active workspace
├── list      List experiments in the active workspace
└── get       Print one experiment's runs and their metrics
```

## Commands

### `mlx experiment create <spec-file>`

**Description:** Creates an experiment in the active workspace from the YAML spec file `<spec-file>` (name, optional one-line description) and reports the created experiment (FR-1).
The spec is validated locally first (FR-4); name uniqueness is confirmed by the server (FR-5).

**Synopsis:**

```
mlx experiment create [options] <spec-file>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<spec-file>` | Yes | Path to the experiment spec YAML file (see specification.md, Data Models) |

**Options:** global options only (see Global Options).

**Examples:**

```bash
mlx experiment create experiment.yaml
mlx experiment create experiment.yaml --json
```

**Errors:**

- `<spec-file>` missing or unreadable: exit 1 naming the file.
- YAML parse failure: exit 1 naming the parse error and the file; no server call (FR-4).
- `name` absent or not kebab-case: exit 1 explaining the rule; no server call (FR-4).
- `description` present but not a single line: exit 1 explaining the one-line rule; no server call (FR-4).
- Duplicate name (server 409): exit 1 with a conflict error naming the experiment (FR-5).
- Authentication failure: exit 2.
- Server unreachable after retries or 5xx: exit 3 with a clear message (NFR-2).

### `mlx experiment list`

**Description:** Lists the experiments in the active workspace, each with its name and its one-line description when present (FR-2).
Serves JSON output per the global `--json` convention.

**Synopsis:**

```
mlx experiment list [options]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:** global options only.

**Examples:**

```bash
mlx experiment list
mlx experiment list --json
```

**Errors:**

- Authentication failure: exit 2.
- Server unreachable after retries: exit 3 with a clear message (NFR-2).

### `mlx experiment get <name>`

**Description:** Prints one experiment's details (name, description) and its runs, each run with its identifier and its metrics from the server's run records (FR-3).
An experiment with no runs prints its details and an empty run listing, exit 0.
Serves JSON output per the global `--json` convention.

**Synopsis:**

```
mlx experiment get [options] <name>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<name>` | Yes | Experiment name; kebab-case string created by `experiment create` |

**Options:** global options only.

**Examples:**

```bash
mlx experiment get ctr-baseline
mlx experiment get ctr-baseline --json
```

**Errors:**

- Missing `<name>`: exit 1 with a usage error.
- Unknown experiment (server 404): exit 1 with an error naming the experiment, suggesting `mlx experiment list` (FR-5).
- Authentication failure: exit 2.
- Server unreachable after retries or 5xx: exit 3 with a clear message (NFR-2).

## Global Options

Owned by FEAT-p1 and available on every command; listed here because this group relies on them.

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--profile` | `<name>` | default profile | Use a named configuration profile; the active workspace is read from this profile |
| | `--json` | | human output | Emit a single JSON document to stdout and nothing else |
| | `--verbose` | | off | Informational logging to stderr, never stdout |
| | `--version` | | | Print version and exit |
| | `--help` | | | Show help and exit |

## Environment Variables

None defined by this command group.
Profile and server resolution environment variables, if any, are owned by the FEAT-p1 global design.

## Configuration

- The group reads the active workspace from the active profile's stored selection (owned by the workspace group, FEAT-p1 FR-3); it stores nothing.
- Precedence: CLI flags > environment variables > config file > defaults (FEAT-p1 global rule; this group adds no flags or variables).

## Exit Codes

| Code | Meaning | Used by this group for |
|---|---|---|
| 0 | Success | Any successful command, including `get` on an experiment with no runs |
| 1 | Usage error | Missing or unreadable `<spec-file>`; local validation failures; unknown experiment on `get`; duplicate-name conflict on `create` |
| 2 | Authentication failure | Token rejected or expired |
| 3 | Server error | Server unreachable after retries; server 5xx |

## Output Behavior

- **stdout:** the data (confirmation line for `create`, table for `list`, details plus run listing for `get`); nothing else in JSON mode.
- **stderr:** diagnostics, retry notices, and errors.
- **Machine-readable (`--json`):**
  - `create`: the created experiment object, e.g. `{"name": "ctr-baseline", "description": "First pass on CTR estimation"}`; the `description` key is omitted when absent.
  - `list`: an array of experiment objects, e.g. `[{"name": "ctr-baseline", "description": "First pass on CTR estimation"}]`; the `description` key is omitted when absent.
  - `get`: the experiment with nested runs, e.g. `{"name": "ctr-baseline", "description": "First pass on CTR estimation", "runs": [{"id": "ctr-baseline-run", "metrics": {"auc": 0.81, "loss": 0.42}}]}`; `runs` is `[]` when the experiment has none.
  - Unknown response fields are tolerated and ignored (forward compatibility, per the FEAT-p2 contract conventions).
- **TTY behavior:** the `list` table and the `get` run listing render with borders and color only when stdout is a TTY; honors `NO_COLOR`.
  Piped output renders as a plain aligned table (human mode) or JSON (`--json`).

## Help Text

```
Usage: mlx experiment [OPTIONS] COMMAND [ARGS] []

Group training runs into experiments and inspect their metrics.

Commands:
  create  Create an experiment in the active workspace
  list    List experiments in the active workspace
  get     Print one experiment's runs and their metrics
```

Representative subcommand help:

```
Usage: mlx experiment create [OPTIONS] <spec-file>

Create an experiment in the active workspace from a YAML spec.

The spec has a required kebab-case name and an optional
one-line description.
Training runs attach to the experiment by name through the
training spec's experiment field.

Arguments:
  <spec-file>  Path to the experiment spec YAML file
```

## Error Messages

- Format: `error: <message>` on stderr, followed by `hint: <fix>` when a fix exists.
- Name is not kebab-case on `create`:

  ```
  error: experiment name must be kebab-case (lowercase letters, digits, hyphens), got 'CTR_Baseline'
  hint: try 'ctr-baseline'
  ```

- Multi-line description on `create`:

  ```
  error: experiment description must be a single line
  hint: keep the description to one line; put long notes in the training spec or your own docs
  ```

- Duplicate name on `create`:

  ```
  error: experiment 'ctr-baseline' already exists in this workspace
  hint: experiment names are unique per workspace; pick another name or inspect it with `mlx experiment get ctr-baseline`
  ```

- Unknown experiment on `get`:

  ```
  error: experiment 'ctr-basline' does not exist in this workspace
  hint: list experiments with `mlx experiment list`
  ```

## Interactive Behavior

- **Prompts:** none; every command is non-interactive.
- **Destructive actions:** none; the group has no delete or update commands in v1, so no confirmations are needed.
- **Dry run:** not applicable.

## Example Sessions

### Happy path: create, attach runs, inspect

```console
$ cat experiment.yaml
name: ctr-baseline
description: First pass on CTR estimation
$ mlx experiment create experiment.yaml
experiment 'ctr-baseline' created
$ mlx job submit train.yaml
job ctr-baseline-run submitted (experiment: ctr-baseline)
$ mlx experiment list
NAME             DESCRIPTION
ctr-baseline     First pass on CTR estimation
$ mlx experiment get ctr-baseline
experiment: ctr-baseline
description: First pass on CTR estimation
runs:
  1. ctr-baseline-run
     auc=0.81 loss=0.42
```

### Error path: spec rejected locally, no server call

```console
$ cat experiment.yaml
name: CTR_Baseline
$ mlx experiment create experiment.yaml
error: experiment name must be kebab-case (lowercase letters, digits, hyphens), got 'CTR_Baseline'
hint: try 'ctr-baseline'
$ echo $?
1
```

### Error path: unknown experiment on get

```console
$ mlx experiment get ctr-basline
error: experiment 'ctr-basline' does not exist in this workspace
hint: list experiments with `mlx experiment list`
$ echo $?
1
```

### Piping example

```console
$ mlx experiment list --json | jq -r '.[].name'
ctr-baseline
```

## Requirements Traceability

| Requirement | Command(s) / Option(s) | Notes |
|---|---|---|
| FR-1 | `mlx experiment create <spec-file>` | Local validation, then server create; reports the created name |
| FR-2 | `mlx experiment list` | Table of name and description |
| FR-3 | `mlx experiment get <name>` | Details plus run listing with identifiers and metrics; JSON shape defined under Output Behavior |
| FR-4 | `mlx experiment create` validation rules | YAML parse, name presence, kebab-case; exit 1 with no server call |
| FR-5 | error paths on `create` and `get` | 409 conflict and 404 not-found mappings under Exit Codes and Error Messages |
| NFR-1 | help text + error format | Every command has help; every error carries cause and hint |
| NFR-2 | retry policy on all three commands | Inherited from the FEAT-p1 client core; one informational retry line |
| NFR-3 | `mlx experiment list`, `mlx experiment get` | Rendering budget under 500 ms warm |
| NFR-4 | (toolchain gate) | Not a command; verified repository-wide |

## Out of Scope

- Creating, modifying, or deleting runs: runs are created by training jobs (FEAT-p9) and their metrics by the server (FEAT-p2).
- Parsing run logs in the CLI for metrics (open question 1 default: server run records only).
- Experiment deletion, renaming, or archive: not in the parent command surface (FEAT-p1 plan).
- Auto-creating an experiment referenced by a training spec at submit time: decision owned at the FEAT-p9 boundary (open question 2).

## Open Questions

1. Do run metrics come from the server's run records only, or can the CLI also parse run logs? (v1 default here: server run records only.)
2. Does submitting a training spec that references a never-created experiment name auto-create the experiment or fail? (v1 default here: fail naming the experiment; shared with FEAT-p9.)
3. Must `experiment get` paginate runs for experiments with many runs, or does the server return all runs of one experiment in a single response in v1? (Design assumes a single response; revisit when the FEAT-p2 experiment endpoints land.)

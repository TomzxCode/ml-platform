---
title: "mlx job command"
status: draft
parent: FEAT-p1
---

# CLI Design: mlx job command

## Design Principles

- Extends the `mlx` CLI conventions fixed by the FEAT-p1 plan: cyclopts, kebab-case command names, verb-first subcommands under a noun group.
- Global options, exit code scheme, and output modes are inherited from FEAT-p1 and are restated here only where the job group specializes them.
- Local validation before submission: every spec problem the CLI can detect without the server is reported before any network call, naming the offending field and the fix.
- The server is authoritative: job state, reference existence, placement, and log retention are server-side; the CLI reports what the server returns and never derives state itself.
- Streaming is a first-class mode: `job logs --follow` holds the stream open and prints lines as they arrive, with interrupt as a supported ending, not a failure.
- The job spec schema this group validates is the normative schema owned by this feature; FEAT-p10's `batch submit` consumes the same module (FR-8).

## Command Tree

```
mlx
└── job                 Submit and track jobs (processing, training, batch inference)
    ├── cancel          Request cancellation of a job
    ├── get             Print one job's current state and details
    ├── list            List jobs in the workspace
    ├── logs            Print a job's logs, optionally following them
    └── submit          Submit a job defined by a spec file
```

## Commands

### `mlx job submit <spec-file>`

**Description:** Validates the spec file locally (YAML parse, required fields per `type`, enum values, structural reference checks), submits it to the server, and reports the server-assigned job id (FR-1, FR-2, FR-3).
Secret references are names only and their values never transit the CLI (FR-9).
The submission carries an idempotency key so a client-side retry never creates a second job (FR-10).

**Synopsis:**

```
mlx job submit <spec-file>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<spec-file>` | Yes | Path to the YAML job spec discriminated by `type` (`processing`, `training`, or `batch-inference`) |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx job submit training-spec.yaml
mlx job submit nightly-scoring.yaml --json
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<spec-file>` argument | 1 | cyclopts usage error naming the argument |
| Spec file does not exist | 1 | `error: spec file '<path>' does not exist` |
| YAML parse failure | 1 | `error: '<path>' is not valid YAML: <parse error>` |
| Missing required field | 1 | `error: missing required field '<field>' for type <type>` plus hint |
| Invalid `type` or `--type` enum value | 1 | error listing the accepted values |
| Malformed reference (model without version, non-kebab-case name) | 1 | error naming the field and the expected form |
| Unknown dataset, model, secret, or compute type (server-confirmed) | 1 | `error: unknown <kind> '<name>'` plus the server's suggestion when present |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint to run `mlx auth login` |
| Server unreachable after backoff | 3 | `error: cannot reach the API server at <url>` plus hint |

**Local validation order** (fixed so the first failure is deterministic): file exists, YAML parses, `type` is present and valid, required common fields, required per-type fields in table order, `compute.type` form, structural reference forms (`name:version` for models, kebab-case for dataset, secret, and compute names).

### `mlx job list`

**Description:** Lists the jobs in the active workspace with id, name, type, and state, optionally filtered by `--type` and `--state` (FR-4).
The listing follows the server's cursor pagination to exhaustion, so the printed result is always complete.

**Synopsis:**

```
mlx job list [--type <type>] [--state <state>]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |
| | `--type` | `processing` \| `training` \| `batch-inference` | all types | Filter the listing to one job type |
| | `--state` | `queued` \| `running` \| `succeeded` \| `failed` \| `cancelled` | all states | Filter the listing to one job state |

**Examples:**

```bash
mlx job list
mlx job list --type training --state running
mlx job list --json | jq -r '.[] | select(.state == "failed") | .id'
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Invalid `--type` value | 1 | error listing the accepted types |
| Invalid `--state` value | 1 | error listing the accepted states |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as submit |

**Empty result:** exits 0 and prints a table with no rows in human mode, an empty JSON array in `--json` mode.

### `mlx job get <id>`

**Description:** Prints one job's current state and details as a key-value block: id, name, type, state, compute type and count, creation time, and the type-specific fields (datasets, model, experiment) (FR-5).
Secret references print as names; `env` prints keys only in v1 (open question 6 in `requirements.md`).

**Synopsis:**

```
mlx job get <id>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<id>` | Yes | Job id as printed by `job submit` or `job list`; opaque server-assigned string |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx job get j-4yd7
mlx job get j-4yd7 --json
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<id>` argument | 1 | cyclopts usage error naming the argument |
| Unknown job id | 1 | `error: no job with id '<id>'` plus hint to run `job list` |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as submit |

### `mlx job logs <id> [--follow]`

**Description:** Prints a job's accumulated log lines to stdout and exits (FR-6).
With `--follow`, the stream stays open and prints lines as the job produces them, ending when the job reaches a terminal state or the user interrupts.
Interrupting a follow stream is a supported ending and exits 0 (decision below; open question 3).

**Synopsis:**

```
mlx job logs <id> [--follow]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<id>` | Yes | Job id as printed by `job submit` or `job list` |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |
| | `--follow` | | off | Keep the stream open, printing lines as they are produced |

**Examples:**

```bash
mlx job logs j-4yd7
mlx job logs j-4yd7 --follow
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<id>` argument | 1 | cyclopts usage error naming the argument |
| Unknown job id | 1 | `error: no job with id '<id>'` plus hint to run `job list` |
| `--json` combined with `--follow` | 1 | `error: --json cannot follow a live log stream` plus hint to drop `--follow` |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as submit |

### `mlx job cancel <id>`

**Description:** Requests cancellation of one job and prints a confirmation line (FR-7).
Cancellation of a job already in a terminal state is refused with an error naming the current state.
No confirmation prompt: a cancelled job is resubmittable from the same spec, so the action is recoverable (default recorded as an open question for the parent surface owner).

**Synopsis:**

```
mlx job cancel <id>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<id>` | Yes | Job id as printed by `job submit` or `job list` |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx job cancel j-4yd7
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<id>` argument | 1 | cyclopts usage error naming the argument |
| Unknown job id | 1 | `error: no job with id '<id>'` plus hint to run `job list` |
| Job already terminal | 1 | `error: job '<id>' already <state>` plus hint that only queued or running jobs can be cancelled |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as submit |

## Global Options

Inherited from FEAT-p1; the job group consumes them as follows.

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--profile` | name | `default` | Selects the profile whose selected workspace scopes all job operations |
| | `--json` | | off | Single JSON document on stdout, nothing else (see Output Behavior for `logs`) |
| | `--verbose` | | off | Informational logging to stderr |
| | `--version` | | | Print version and exit |
| -h | `--help` | | | Per-command help |

## Environment Variables

| Variable | Used by | Default | Description |
|---|---|---|---|
| `NO_COLOR` | all | unset | Disables color on TTY output |

## Configuration

- No job-specific configuration and no local state; the group writes nothing outside the terminal.
- Workspace scoping comes from the active profile's selected workspace (`mlx workspace select`); precedence: CLI flags > profile config > defaults.

## Exit Codes

Inherited from the FEAT-p1 plan.

| Code | Meaning |
|---|---|
| 0 | Success (including an empty listing, empty logs, and a user-interrupted follow stream) |
| 1 | Usage error (missing or malformed argument, validation failure, unknown resource, terminal-state cancel) |
| 2 | Authentication failure |
| 3 | Server error (unreachable after backoff) |
| 4 | Cancelled by user (not used by this group in v1; interrupting a follow stream exits 0 by design) |

## Output Behavior

- **stdout:** the result only; a table for `list`, a key-value block for `get`, raw log lines for `logs`, one confirmation line for `submit` and `cancel`.
- **stderr:** diagnostics, retry lines, and the follow-mode attachment note; never log content.
- **Machine-readable:** `--json` prints one document per command.
  `job submit` emits `{"id", "name", "type", "state"}`; `job list` emits an array of `{"id", "name", "type", "state"}` objects; `job get` emits one job object with the same fields plus compute and type-specific fields; `job cancel` emits `{"id", "state"}`.
  `job logs` without `--follow` emits one JSON array of line strings; `--json --follow` is rejected (see Errors).
- **Job object shape (design intent, pending the FEAT-p2 contract):** `{"id", "name", "type", "state", "compute": {"type", "count"}, "created_at", ...}` with type-specific fields (`inputs`/`outputs`, `dataset`, `model`, `experiment`, `secrets` as names) and `env` keys only; unknown response fields are tolerated and ignored.
- **Human table columns:** ID, NAME, TYPE, STATE.
- **TTY behavior:** table borders and color on a TTY, plain output otherwise; honors `NO_COLOR`; identical content piped or redirected.

## Help Text

```
Usage: mlx job [COMMAND]

Submit and track jobs: processing, training, and batch inference.

Commands:
  cancel  Request cancellation of a job
  get     Print one job's current state and details
  list    List jobs in the workspace
  logs    Print a job's logs, optionally following them
  submit  Submit a job defined by a spec file
```

Representative subcommand help:

```
Usage: mlx job submit <spec-file>

Submit a job defined by a YAML spec whose type field selects processing,
training, or batch inference. The spec is validated locally before any
server call; each failure names the offending field.
```

## Error Messages

- Format: `error: <message>` then `hint: <fix>` on the next line (NFR-1).
- Representative examples:

```
error: missing required field 'dataset' for type training
hint: add dataset: <catalog-name> to the spec, or run mlx dataset list to see names
```

```
error: unknown dataset 'clened-clicks'
hint: run mlx dataset list to see catalog names in this workspace
```

```
error: job 'j-4yd7' already succeeded
hint: only queued or running jobs can be cancelled; resubmit from the spec instead
```

## Interactive Behavior

- **Prompts:** none; all five subcommands are non-interactive and safe in scripts and CI.
- **Destructive actions:** `job cancel` is the only mutating action besides submission; it carries no confirmation prompt in v1 because a cancelled job is recoverable by resubmitting the same spec (open question 5).
- **Interrupt:** Ctrl-C during `job logs --follow` closes the stream cleanly and exits 0; the job keeps running.
- **Dry run:** not offered in v1 (open question 1).

## Example Sessions

### Submitting a training spec and tracking it

```console
$ mlx job submit training-spec.yaml
submitted j-4yd7 (training, queued)
$ mlx job list
ID       NAME                TYPE       STATE
j-4yd7   ctr-baseline-run    training   queued
j-4a21   nightly-sanitize    processing running
$ mlx job get j-4yd7
Id:         j-4yd7
Name:       ctr-baseline-run
Type:       training
State:      running
Compute:    gpu-a100-40g x 2
Created:    2026-08-23T14:02:11Z
Dataset:    cleaned-clicks
Experiment: ctr-baseline
Secrets:    wandb-key
$ mlx job logs j-4yd7 --follow
epoch 1/10 loss=0.612
epoch 2/10 loss=0.544
^C
$ echo $?
0
```

### Local validation failure, no server call

```console
$ mlx job submit training-spec.yaml
error: missing required field 'dataset' for type training
hint: add dataset: <catalog-name> to the spec, or run mlx dataset list to see names
$ echo $?
1
```

### Server-side reference failure

```console
$ mlx job submit training-spec.yaml
error: unknown dataset 'clened-clicks'
hint: run mlx dataset list to see catalog names in this workspace
$ echo $?
1
```

### Machine-readable consumption

```console
$ mlx job list --json | jq -r '.[] | select(.state == "failed") | .id'
j-3c8f1
$ mlx job cancel j-3c8f1
cancelled j-3c8f1
```

## Requirements Traceability

| Requirement | Command(s) / Option(s) | Notes |
|---|---|---|
| FR-1 | `mlx job submit <spec-file>` | Local validation, then submission, then job id reported |
| FR-2 | `job submit` local validation order | Deterministic order; every failure names the field; no server call |
| FR-3 | `job submit` error table | Server-confirmed unknown references named with suggestions |
| FR-4 | `mlx job list`, `--type`, `--state` | Complete listing via cursor following; local enum checks on both flags |
| FR-5 | `mlx job get <id>` | Key-value block; 404 surfaced naming the id |
| FR-6 | `mlx job logs <id>`, `--follow` | Raw lines; follow until terminal state or interrupt |
| FR-7 | `mlx job cancel <id>` | Confirmation line; terminal-state refusal names the state |
| FR-8 | `job submit` schema module | The validated schema is the shared, normative module FEAT-p10 consumes |
| FR-9 | `job submit`, `job get` output | Secret names only; values never transit the CLI |
| FR-10 | `job submit` idempotency key | Retried submission replays the server's stored response |
| NFR-1 | all commands | `error:` plus `hint:` format on every failure path |
| NFR-2 | all commands | Backoff retry then exit 3 |
| NFR-3 | `job submit`, `job get` | No secret values or credentials in any output mode |
| NFR-4 | `job list`, `job get` | Rendering under 500 ms excluding server time; `logs --follow` excluded |
| NFR-5 | (toolchain) | Covered by the repository gates and argument-surface tests |

## Out of Scope

- The `batch` fast path commands (owned by FEAT-p10); this group provides only the shared spec schema module.
- Server-side scheduling, placement, quota enforcement, and log retention (owned by FEAT-p2).
- Experiment creation and run metrics views (owned by FEAT-p11); this group only submits the `experiment` reference.
- Dataset catalog management (owned by FEAT-p5), secret management (FEAT-p7), and compute type listing (FEAT-p8).
- Log download to a file, time filtering, and tail limits; add flags only on demand via a parent-surface change.

## Open Questions

1. Should `job submit` offer `--dry-run` (validate only, no submission), or defer it pending demand?
2. Should `--json --follow` stream newline-delimited JSON events instead of being rejected (current default: usage error, since the global JSON contract requires exactly one document)?
3. Is exit 0 on interrupting a follow stream the right cross-group convention (it applies to `batch logs` in FEAT-p10 and any future watch-style command)?
4. What is the job id shape and can the CLI validate its form locally, or must ids stay fully opaque until the FEAT-p2 contract lands?
5. Should `job cancel` carry a `--yes` confirmation flag if the parent surface owner mandates prompts on mutating commands (current default: no prompt)?
6. Does `job get` echo `env` values (current default: keys only) and what does the JSON object carry for `env` (mirrors requirements Open Question 6)?
7. Does `job logs --follow` on a queued job hold the stream open until the first line or terminal state, or exit with a note (mirrors requirements Open Question 5)?

---
title: "mlx deploy command"
status: draft
parent: FEAT-p1
---

# CLI Design: mlx deploy command

## Design Principles

- Extends the `mlx` CLI conventions fixed by the FEAT-p1 plan: cyclopts, kebab-case command names, verb-first subcommands under a noun group.
- Global options, exit code scheme, and output modes are inherited from FEAT-p1 and are restated here only where the deploy group specializes them.
- Flags are the whole input surface: this group submits no spec file, so every input arrives as a model reference plus scaling flags, and every rule the CLI can check without the server is checked before any network call.
- Scaling inputs are the footgun surface: the mutual exclusion of `--replicas` and the autoscaling bounds, and the min less than or equal to max rule, fail fast locally with the rule named (FEAT-p1-FR-10 scenarios).
- The server is authoritative: deployment state, rollout execution, endpoint provisioning, and resource release are server-side; the CLI reports what the server returns and never derives state (rendered verbatim, unknown values included).
- One name addresses everything in v1: the endpoint name (explicit via `--endpoint-name` or generated) also names the deployment for `deploy get`, `deploy update`, and `deploy stop` (open question 4 records the confirmation risk).
- `deploy create` returns when the server accepts the request, not when the endpoint is ready; readiness is observed via `deploy get` (open question 3).

## Command Tree

```
mlx
└── deploy              Manage online inference deployments
    ├── create          Deploy a model version for online inference
    ├── get             Print one deployment's state and details
    ├── list            List deployments in the workspace
    ├── stop            Stop a deployment and release its resources
    └── update          Update the served model and/or scaling
```

## Commands

### `mlx deploy create <model>:<version>`

**Description:** Deploys a registered model version for online inference and reports the endpoint reference plus the deployment state (FR-1, FR-2, FR-3).
Scaling comes either from a static `--replicas` count or from `--min-replicas`/`--max-replicas` bounds, never both.
The submission carries an idempotency key so a client-side retry never creates a second deployment (FR-9).
The command returns on server acceptance; the endpoint becomes ready server-side and `deploy get` shows when it serves.

**Synopsis:**

```
mlx deploy create <model>:<version> [--endpoint-name <name>] [--replicas <n>] [--min-replicas <n>] [--max-replicas <n>]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<model>:<version>` | Yes | Registered model reference, kebab-case name plus version (numeric per the p9 schema convention); existence confirmed server-side |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |
| | `--endpoint-name` | kebab-case name | generated | Name for the endpoint; also names the deployment in v1 (one-name design) |
| | `--replicas` | positive integer | 1 | Static replica count; mutually exclusive with the bound flags |
| | `--min-replicas` | positive integer | unset | Autoscale lower bound; requires or implies `--max-replicas` |
| | `--max-replicas` | positive integer | unset | Autoscale upper bound; must be greater than or equal to min |

Omitting every scaling flag deploys with a static count of 1.

**Examples:**

```bash
mlx deploy create ctr-estimator:2
mlx deploy create ctr-estimator:2 --endpoint-name fraud-scorer --min-replicas 2 --max-replicas 8
mlx deploy create ctr-estimator:2 --replicas 3 --json
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<model>:<version>` argument | 1 | cyclopts usage error naming the argument |
| Malformed model reference | 1 | `error: '<ref>' is not a valid model reference (expected name:version)` plus hint to `mlx model list` |
| `--replicas` combined with a bound flag | 1 | error naming the mutual exclusion rule |
| min greater than max | 1 | error naming the min and max rule |
| Non-positive or non-integer count | 1 | error naming the positive-integer rule |
| `--endpoint-name` not kebab-case | 1 | error naming the expected form |
| Only one bound flag passed | 1 | error naming the bound flag that is missing |
| Unknown model or version (server-confirmed) | 1 | `error: unknown model '<ref>'` plus hint to `mlx model list` |
| Endpoint name already in use (server-confirmed) | 1 | `error: endpoint name '<name>' is already in use` plus hint to choose another |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint to run `mlx auth login` |
| Server unreachable after backoff | 3 | `error: cannot reach the API server at <url>` plus hint |

**Local validation order** (fixed so the first failure is deterministic): argument present, model reference form, scaling flag combination (static versus bounds, bounds paired), min less than or equal to max, positive integers, `--endpoint-name` form.

### `mlx deploy list`

**Description:** Lists the deployments in the active workspace with name, state, served model version, and endpoint reference (FR-4).
The listing follows the server's cursor pagination to exhaustion, so the printed result is always complete.

**Synopsis:**

```
mlx deploy list
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

No filter flags in v1; the parent surface defines none for this group (add via a parent-feature change on demand).

**Examples:**

```bash
mlx deploy list
mlx deploy list --json
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as create |

**Empty result:** exits 0 and prints a table with no rows in human mode, an empty JSON array in `--json` mode.

### `mlx deploy get <name>`

**Description:** Prints one deployment's current state and details as a key-value block: name, state, served model version, scaling configuration (static count or bounds), endpoint reference, and creation time (FR-5).
While the server reports a rollout, the block also carries a rollout line showing the from and to versions and the rollout status (FR-8).

**Synopsis:**

```
mlx deploy get <name>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<name>` | Yes | Deployment name, the endpoint name chosen or generated at create time (one-name design) |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx deploy get fraud-scorer
mlx deploy get fraud-scorer --json
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<name>` argument | 1 | cyclopts usage error naming the argument |
| Unknown deployment | 1 | `error: no deployment '<name>'` plus hint to run `deploy list` |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as create |

### `mlx deploy update <name>`

**Description:** Updates the served model version and/or the scaling of one deployment (FR-6, FR-8).
Any non-empty subset of the flags applies; omitted flags keep their current values.
Setting `--replicas` clears autoscaling; setting bounds replaces a static count.
Updating the model triggers a rollout whose progress is visible via `deploy get`; the command itself returns when the server accepts the update (open question 1 records the streaming alternative).

**Synopsis:**

```
mlx deploy update <name> [--model <model>:<version>] [--replicas <n>] [--min-replicas <n>] [--max-replicas <n>]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<name>` | Yes | Deployment name |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |
| | `--model` | `name:version` | keep current | New served model version; existence confirmed server-side |
| | `--replicas` | positive integer | keep current | New static replica count; clears autoscaling |
| | `--min-replicas` | positive integer | keep current | New autoscale lower bound; replaces a static count |
| | `--max-replicas` | positive integer | keep current | New autoscale upper bound |

**Examples:**

```bash
mlx deploy update fraud-scorer --model ctr-estimator:3
mlx deploy update fraud-scorer --replicas 3
mlx deploy update fraud-scorer --min-replicas 2 --max-replicas 8
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<name>` argument | 1 | cyclopts usage error naming the argument |
| No update flag passed | 1 | `error: nothing to update` plus hint naming at least one of the four flags |
| Malformed `--model` reference | 1 | same shape as create |
| Scaling rule violation (mutual exclusion, bounds pairing, min greater than max, non-positive) | 1 | same shapes as create |
| Unknown deployment | 1 | `error: no deployment '<name>'` plus hint to run `deploy list` |
| Unknown model or version (server-confirmed) | 1 | same shape as create |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as create |

An update that passes only one bound flag is a usage error, matching create.

### `mlx deploy stop <name>`

**Description:** Requests that the server stop the deployment and release its resources, and prints a confirmation (FR-7).
Stopping a deployment that is already stopped is refused with an error naming the current state.
No confirmation prompt: a stopped model is one `deploy create` away from serving again, so the action is recoverable (open question 2 records the escalation).

**Synopsis:**

```
mlx deploy stop <name>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<name>` | Yes | Deployment name |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx deploy stop fraud-scorer
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<name>` argument | 1 | cyclopts usage error naming the argument |
| Unknown deployment | 1 | `error: no deployment '<name>'` plus hint to run `deploy list` |
| Deployment already stopped | 1 | `error: deployment '<name>' is already stopped` plus hint to run `deploy create` to serve again |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as create |

## Global Options

Inherited from FEAT-p1; the deploy group consumes them as follows.

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--profile` | name | `default` | Selects the profile whose selected workspace scopes all deploy operations |
| | `--json` | | off | Single JSON document on stdout, nothing else |
| | `--verbose` | | off | Informational logging to stderr |
| | `--version` | | | Print version and exit |
| -h | `--help` | | | Per-command help |

## Environment Variables

| Variable | Used by | Default | Description |
|---|---|---|---|
| `NO_COLOR` | all | unset | Disables color on TTY output |

## Configuration

- No deploy-specific configuration and no local state; the group writes nothing outside the terminal.
- Workspace scoping comes from the active profile's selected workspace (`mlx workspace select`); precedence: CLI flags > profile config > defaults.

## Exit Codes

Inherited from the FEAT-p1 plan.

| Code | Meaning |
|---|---|
| 0 | Success (including an empty listing) |
| 1 | Usage error (missing or malformed argument, validation failure, unknown resource, stop of an already-stopped deployment) |
| 2 | Authentication failure |
| 3 | Server error (unreachable after backoff) |
| 4 | Cancelled by user (not used by this group in v1; no interactive prompts or streams) |

## Output Behavior

- **stdout:** the result only; a confirmation line plus endpoint reference for `create`, a table for `list`, a key-value block for `get`, a confirmation line for `update` and `stop`.
- **stderr:** diagnostics and retry lines; never results.
- **Machine-readable:** `--json` prints one document per command.
  `deploy create` emits one deployment object; `deploy list` emits an array of summaries `{"name", "state", "model", "endpoint"}`; `deploy get` emits one deployment object (plus `rollout` when the server reports one); `deploy update` emits the updated deployment object; `deploy stop` emits `{"name", "state"}`.
- **Deployment object shape (design intent, pending the FEAT-p2 contract):** `{"name", "endpoint", "model", "state", "scaling", "created_at"}` where `model` is `name:version`, `scaling` is `{"mode": "static", "replicas": n}` or `{"mode": "autoscale", "min_replicas": n, "max_replicas": n}`, and `rollout`, when present, is `{"from_model", "to_model", "status"}`; unknown response fields are tolerated and ignored.
- **Human table columns:** NAME, STATE, MODEL, ENDPOINT.
- **TTY behavior:** table borders and color on a TTY, plain output otherwise; honors `NO_COLOR`; identical content piped or redirected.

## Help Text

```
Usage: mlx deploy [COMMAND]

Manage online inference deployments.

Commands:
  create  Deploy a model version for online inference
  get     Print one deployment's state and details
  list    List deployments in the workspace
  stop    Stop a deployment and release its resources
  update  Update the served model and/or scaling
```

Representative subcommand help:

```
Usage: mlx deploy create <model>:<version> [OPTIONS]

Deploy a registered model version for online inference and print the
endpoint reference. Scaling is either a static replica count or autoscale
bounds, never both; every rule is checked locally before any server call.
```

## Error Messages

- Format: `error: <message>` then `hint: <fix>` on the next line (NFR-1).
- Representative examples:

```
error: --replicas cannot be combined with --min-replicas/--max-replicas
hint: pass either a static count or autoscaling bounds, not both
```

```
error: unknown model 'ctr-estimator:9'
hint: run mlx model list to see registered models and versions
```

```
error: no deployment 'fraud-scorer'
hint: run mlx deploy list to see deployments in this workspace
```

## Interactive Behavior

- **Prompts:** none; all five subcommands are non-interactive and safe in scripts and CI.
- **Destructive actions:** `deploy stop` is the destructive action; it carries no confirmation prompt in v1 because serving again is one `deploy create` away (open question 2).
- **Interrupt:** no long-running commands in v1; Ctrl-C during a request aborts with the parent's cancelled-by-user handling.
- **Dry run:** not offered; local validation always runs before any server call, which is the whole benefit a dry run would add for a flag-only surface.

## Example Sessions

### Deploy, inspect, and update

```console
$ mlx deploy create ctr-estimator:2 --endpoint-name fraud-scorer --min-replicas 2 --max-replicas 8
deployed ctr-estimator:2 at fraud-scorer (creating)
$ mlx deploy list
NAME           STATE    MODEL            ENDPOINT
fraud-scorer   serving  ctr-estimator:2  fraud-scorer
$ mlx deploy get fraud-scorer
Name:      fraud-scorer
State:     serving
Model:     ctr-estimator:2
Scaling:   autoscale 2-8
Endpoint:  fraud-scorer
Created:   2026-08-23T14:02:11Z
$ mlx deploy update fraud-scorer --model ctr-estimator:3
updating fraud-scorer to ctr-estimator:3 (rollout in progress)
$ mlx deploy get fraud-scorer
Name:      fraud-scorer
State:     updating
Model:     ctr-estimator:3
Rollout:   ctr-estimator:2 -> ctr-estimator:3, in progress
Scaling:   autoscale 2-8
Endpoint:  fraud-scorer
Created:   2026-08-23T14:02:11Z
```

### Local validation failure, no server call

```console
$ mlx deploy create ctr-estimator:2 --replicas 3 --min-replicas 1
error: --replicas cannot be combined with --min-replicas/--max-replicas
hint: pass either a static count or autoscaling bounds, not both
$ echo $?
1
```

### Machine-readable consumption

```console
$ mlx deploy list --json | jq -r '.[] | select(.state == "serving") | .endpoint'
fraud-scorer
$ mlx deploy stop fraud-scorer
stopped fraud-scorer
```

## Requirements Traceability

| Requirement | Command(s) / Option(s) | Notes |
|---|---|---|
| FR-1 | `mlx deploy create <model>:<version>`, `--endpoint-name` | Local validation, then submission, then endpoint reference reported |
| FR-2 | `deploy create`, `deploy update` local validation order | Deterministic order; every failure names the flag and the rule; no server call |
| FR-3 | `deploy create`, `deploy update` error tables | Local `name:version` form check; server-confirmed unknown references named with suggestions |
| FR-4 | `mlx deploy list` | Complete listing via cursor following; empty listing exits 0 |
| FR-5 | `mlx deploy get <name>` | Key-value block; unknown deployment named |
| FR-6 | `mlx deploy update <name>`, its four flags | Non-empty subset rule; static count clears autoscaling; bounds replace static |
| FR-7 | `mlx deploy stop <name>` | Confirmation line; already-stopped refusal names the state |
| FR-8 | `deploy update`, `deploy get` rollout line | Rollout progress rendered from server-reported rollout fields |
| FR-9 | `deploy create` idempotency key | Retried create replays the server's stored response |
| NFR-1 | all commands | `error:` plus `hint:` format on every failure path |
| NFR-2 | all commands | Backoff retry then exit 3 |
| NFR-3 | `deploy list`, `deploy get` | Rendering under 500 ms excluding server time |
| NFR-4 | (toolchain) | Covered by the repository gates and argument-surface tests |

## Out of Scope

- Serving execution, rollout mechanics, endpoint provisioning, and resource release (owned by FEAT-p2).
- Model registration and version inspection (owned by FEAT-p13); this group only references `name:version`.
- Rollout log streaming and a watch mode for `deploy get` (open question 1; add via a parent-surface change on demand).
- List filters, tag or description metadata, and traffic splitting between versions; add only on demand via a parent-surface change.

## Open Questions

1. Should rollout progress stream during `deploy update`, or remain visible only via `deploy get` (current default: `get` only; no streaming command in v1)?
2. Should `deploy stop` require confirmation or a `--yes` flag (current default: no prompt; redeploying is one create away)?
3. Does `deploy create` return on server acceptance or wait for endpoint readiness (current default: return on acceptance; readiness via `deploy get`)?
4. Is the one-name design right (the endpoint name also addresses the deployment), or does the server assign a distinct deployment name the CLI must print and accept (mirrors requirements Open Question 4)?
5. Which state values does the server report for deployments and rollouts, and does `updating` with a `rollout` object match the contract (current default: render verbatim, open on read)?
6. Is the version part of a model reference numeric (current default: numeric, following the p9 schema) or arbitrary (mirrors requirements Open Question 6)?

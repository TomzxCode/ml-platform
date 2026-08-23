---
title: "mlx compute command"
status: draft
parent: FEAT-p1
---

# CLI Design: mlx compute command

## Design Principles

- Extends the `mlx` CLI conventions fixed by the FEAT-p1 plan: cyclopts, kebab-case command names, verb-first subcommands under a noun group.
- Global options, exit code scheme, and output modes are inherited from FEAT-p1 and are restated here only where the compute group specializes them.
- Read-only by construction: two commands, no mutations, no prompts, no destructive paths; nothing to confirm and nothing to undo.
- The server is authoritative: the CLI holds no local catalog of machine types, applies only format-level validation locally, and renders unknown-type failures using the server's closest-match suggestion.

## Command Tree

```
mlx
└── compute              Discover compute types available to the workspace
    ├── get              Print one machine type's details and quota
    └── list             List machine types available to the workspace
```

## Commands

### `mlx compute list`

**Description:** Lists the machine types available to the active workspace, each with its GPU model and count, memory, and the workspace's remaining quota for the type (FR-1).
The active workspace is the one selected in the active profile.

**Synopsis:**

```
mlx compute list
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx compute list
mlx compute list --json | jq -r '.[].type'
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Not authenticated or session invalid | 2 | inherited authentication error plus hint to run `mlx auth login` |
| Server unreachable after backoff | 3 | `error: cannot reach the API server at <url>` plus hint |

**Empty result:** exits 0 and prints a table with no rows in human mode, an empty JSON array in `--json` mode.

### `mlx compute get <type>`

**Description:** Prints one machine type's details (type name, GPU model and count, memory) plus the workspace's remaining quota for that type, as a key-value block (FR-2).
Unknown type names are resolved server-side; the CLI only validates the name's form locally (FR-3).

**Synopsis:**

```
mlx compute get <type>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<type>` | Yes | Compute type name, kebab-case (for example `gpu-h100-80gb`); must match an entry from `compute list` |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx compute get gpu-h100-80gb
mlx compute get gpu-h100-80gb --json
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Missing `<type>` argument | 1 | cyclopts usage error naming the argument |
| Malformed type name (not kebab-case) | 1 | `error: '<value>' is not a valid compute type name` plus expected form |
| Unknown (well-formed) type name | 1 | `error: unknown compute type '<name>'` plus the server's closest-match suggestion |
| Not authenticated or session invalid | 2 | inherited authentication error plus hint |
| Server unreachable after backoff | 3 | same shape as list |

**Local validation rule:** a type name matches `^[a-z0-9]+(-[a-z0-9]+)*$` (lowercase alphanumeric segments joined by single hyphens); anything else is rejected locally without contacting the server.

## Global Options

Inherited from FEAT-p1; the compute group consumes them as follows.

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--profile` | name | `default` | Selects the profile whose selected workspace scopes the listing |
| | `--json` | | off | Single JSON document on stdout, nothing else |
| | `--verbose` | | off | Informational logging to stderr |
| | `--version` | | | Print version and exit |
| -h | `--help` | | | Per-command help |

## Environment Variables

| Variable | Used by | Default | Description |
|---|---|---|---|
| `NO_COLOR` | all | unset | Disables color on TTY output |

## Configuration

- No compute-specific configuration and no local state; the group writes nothing.
- Workspace scoping comes from the active profile's selected workspace (`mlx workspace select`); precedence: CLI flags > profile config > defaults.

## Exit Codes

Inherited from the FEAT-p1 plan.

| Code | Meaning |
|---|---|
| 0 | Success (including an empty listing) |
| 1 | Usage error (missing or malformed argument, unknown type name) |
| 2 | Authentication failure |
| 3 | Server error (unreachable after backoff) |
| 4 | Cancelled by user |

## Output Behavior

- **stdout:** the result document only; a table for `list`, a key-value block for `get`.
- **stderr:** diagnostics and errors; retry lines when backoff engages.
- **Machine-readable:** `--json` prints one document per command; `compute list` emits an array of type objects, `compute get` emits a single type object (FR-4).
- **Type object shape (design intent, pending the FEAT-p2 contract):** `{"type", "gpu_model", "gpu_count", "memory_gib", "quota": {"remaining"}}`, with `gpu_model` null and `gpu_count` 0 for CPU-only types and `quota` fields extended (used, limit) if the contract provides them.
- **Human table columns:** TYPE, GPUS (model x count, or `cpu`), MEMORY (GiB), QUOTA (remaining).
- **TTY behavior:** table borders and color on a TTY, plain output otherwise; honors `NO_COLOR`; identical content piped or redirected.

## Help Text

```
Usage: mlx compute [COMMAND]

Discover compute types available to the workspace.

Commands:
  get   Print one machine type's details and quota
  list  List machine types available to the workspace
```

Representative subcommand help:

```
Usage: mlx compute get <type>

Print one machine type's details (GPUs, memory) and the workspace's
remaining quota for it.

Arguments:
  type  Compute type name, kebab-case (see mlx compute list)
```

## Error Messages

- Format: `error: <message>` then `hint: <fix>` on the next line (NFR-1).
- Representative examples:

```
error: 'GPU_H100' is not a valid compute type name
hint: use kebab-case, for example gpu-h100-80gb; run mlx compute list to see available types
```

```
error: unknown compute type 'gpu-a100-80bg'
hint: did you mean 'gpu-a100-80gb'? run mlx compute list to see available types
```

## Interactive Behavior

- **Prompts:** none; both commands are non-interactive and safe in scripts and CI.
- **Destructive actions:** none; the group is read-only.
- **Dry run:** not applicable.

## Example Sessions

### Listing available types

```console
$ mlx compute list
TYPE            GPUS                    MEMORY   QUOTA
cpu-general-8   cpu                     32 GiB   20 of 20
gpu-a100-40gb   NVIDIA A100 40GB x 1    240 GiB  3 of 4
gpu-h100-80gb   NVIDIA H100 80GB x 8    1128 GiB 0 of 2
```

### Inspecting one type

```console
$ mlx compute get gpu-a100-40gb
Type:      gpu-a100-40gb
GPUs:      NVIDIA A100 40GB x 1
Memory:    240 GiB
Quota:     3 of 4 remaining
```

### Machine-readable consumption

```console
$ mlx compute list --json | jq -r '.[].type'
cpu-general-8
gpu-a100-40gb
gpu-h100-80gb
```

### Unknown type with closest match

```console
$ mlx compute get gpu-a100-80bg
error: unknown compute type 'gpu-a100-80bg'
hint: did you mean 'gpu-a100-40gb'? run mlx compute list to see available types
$ echo $?
1
```

### Malformed type name

```console
$ mlx compute get GPU_H100
error: 'GPU_H100' is not a valid compute type name
hint: use kebab-case, for example gpu-h100-80gb; run mlx compute list to see available types
$ echo $?
1
```

## Requirements Traceability

| Requirement | Command(s) / Option(s) | Notes |
|---|---|---|
| FR-1 | `mlx compute list` | Table with type, GPUs, memory, remaining quota; workspace-scoped via the profile |
| FR-2 | `mlx compute get <type>` | Key-value block with details plus quota |
| FR-3 | `mlx compute get`, local validation rule | Local kebab-case check; unknown names surface the server's closest match |
| FR-4 | `--json` on both commands | Array for `list`, single object for `get`, one document only |
| FR-5 | both commands | Exit mapping per the Errors tables |
| NFR-1 | both commands | `error:` plus `hint:` format, closest match named |
| NFR-2 | both commands | Backoff retry then exit 3 |
| NFR-3 | both commands | Startup and rendering under 500 ms excluding server time |
| NFR-4 | (toolchain) | Covered by the repository gates, no CLI surface |

## Out of Scope

- Quota limit administration (owned by the `quota` group, FEAT-p1-FR-21); this group only reports availability.
- Cluster and machine browsing (owned by the `infra` group, FEAT-p1-FR-22).
- List filters such as GPU-only (Open Question 1; deferred pending demand).
- The server-side compute type and quota contract (owned by FEAT-p2).

## Open Questions

1. Should `compute list` support filters (for example GPU-only) in v1, or defer them (mirrors requirements Open Question 1)?
2. If the FEAT-p2 list endpoint paginates, does `compute list` follow cursors transparently, and is that needed at the expected cardinality (mirrors requirements Open Question 2)?
3. What quota fields does the contract return per type (remaining only, or used, limit, and remaining), and do zero-quota types appear in listings (mirrors requirements Open Question 3)?
4. Should the human table's QUOTA column show remaining only or used-of-total (the examples show used-of-total with remaining implied)?

---
title: "mlx workspace command"
status: draft
parent: FEAT-p1
---

# CLI Design: mlx workspace command

## Design Principles

- Extends the `mlx` CLI conventions fixed by the FEAT-p1 plan: kebab-case command names, verb-first subcommands under a noun group, GNU-style long options, cyclopts as the framework.
- Exit codes follow the CLI-wide scheme (0 success, 1 usage error, 2 authentication failure, 3 server error, 4 cancelled by user).
- Human output on stdout, diagnostics on stderr; one valid JSON document on stdout when `--json` is passed and nothing else.
- The workspace roster is authoritative on the server; the CLI validates only locally checkable rules (kebab-case shape, argument presence) before calling the server.
- `get` reads local profile state and makes no network call, so it is fast and works offline.

## Command Tree

```
mlx workspace
├── list      List workspaces the user can access
├── select    Persist the active workspace in the active profile
└── get       Print the active workspace
```

## Commands

### `mlx workspace list`

**Description:** Lists the workspaces the authenticated user can access, with name and description (FR-1).
Serves JSON output per FR-6.

**Synopsis:**

```
mlx workspace list [options]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:** global options only (see Global Options).

**Examples:**

```bash
mlx workspace list
mlx workspace list --json
```

**Errors:**

- Authentication failure: exit 2.
- Server unreachable after retries: exit 3 with a clear message (NFR-2).

### `mlx workspace select <name>`

**Description:** Selects the workspace `<name>` as the active workspace and persists the selection in the active profile, so later commands inherit it (FR-2, FR-4).
Existence and access are validated by the server; an unknown name exits 3 with the closest match suggested (FR-5).

**Synopsis:**

```
mlx workspace select [options] <name>
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| `<name>` | Yes | Workspace name; kebab-case string that must match an entry from `workspace list` |

**Options:** global options only.

**Examples:**

```bash
mlx workspace select research
```

**Errors:**

- Missing `<name>` or name not in kebab-case form: exit 1 with a usage error naming the expected form.
- Unknown name (server rejects): exit 3, message suggests the closest matching workspace name computed from the accessible list.
- Authentication failure: exit 2.

### `mlx workspace get`

**Description:** Prints the active workspace, or a clear statement that none is selected (FR-3).
Reads the active profile's stored selection; makes no network call.
Serves JSON output per FR-6.

**Synopsis:**

```
mlx workspace get [options]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:** global options only.

**Examples:**

```bash
mlx workspace get
mlx workspace get --json
```

**Errors:**

- No selection is not an error: the command exits 0 and clearly states that no workspace is selected (human mode) or prints `null` (JSON mode).
- Malformed profile state (stored selection unreadable): exit 1 naming the profile file and the fix.

## Global Options

Owned by FEAT-p1 and available on every command; listed here because this group relies on them.

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--profile` | `<name>` | default profile | Use a named configuration profile; the workspace selection is read from and written to this profile |
| | `--json` | | human output | Emit a single JSON document to stdout and nothing else (FR-6) |
| | `--verbose` | | off | Informational logging to stderr, never stdout |
| | `--version` | | | Print version and exit |
| | `--help` | | | Show help and exit |

## Environment Variables

None defined by this command group.
Profile and server resolution environment variables, if any, are owned by the FEAT-p1 global design.

## Configuration

- The selection is stored as a `workspace` key in the active profile's section of the CLI configuration file (format and location owned by the FEAT-p1 profile design).
- The selection is per-profile: switching profiles switches the workspace scope with it.
- Precedence: CLI flags > environment variables > config file > defaults (FEAT-p1 global rule; this group adds no flags or variables that override the stored selection).

## Exit Codes

| Code | Meaning | Used by this group for |
|---|---|---|
| 0 | Success | Any successful command, including `get` reporting no selection |
| 1 | Usage error | Missing or malformed `<name>`; malformed profile state |
| 2 | Authentication failure | Token rejected or expired on `list`/`select` |
| 3 | Server error | Server unreachable after retries; server rejects an unknown workspace name |

## Output Behavior

- **stdout:** the data (table for `list`, the active name or no-selection statement for `get`, a confirmation line for `select`); nothing else in JSON mode.
- **stderr:** diagnostics, retry notices, and errors.
- **Machine-readable (`--json`):**
  - `list`: an array of workspace objects, e.g. `[{"name": "research", "description": "Research team workspace"}]`.
  - `select`: the newly selected workspace object, e.g. `{"name": "research", "description": "Research team workspace"}`.
  - `get`: the active workspace object, or `null` when none is selected.
- **TTY behavior:** the `list` table renders with borders and color only when stdout is a TTY; honors `NO_COLOR`.
  Piped output renders as a plain aligned table (human mode) or JSON (`--json`).

## Help Text

```
Usage: mlx workspace [OPTIONS] COMMAND [ARGS] []

Select the workspace that scopes every subsequent operation.

Commands:
  list    List workspaces the user can access
  select  Persist the active workspace in the active profile
  get     Print the active workspace
```

Representative subcommand help:

```
Usage: mlx workspace select [OPTIONS] <name>

Persist the active workspace in the active profile.

The name must match an entry from `mlx workspace list`.
Later commands run in the selected workspace.

Arguments:
  <name>  Workspace name (kebab-case)
```

## Error Messages

- Format: `error: <message>` on stderr, followed by `hint: <fix>` when a fix exists.
- Operation command with no workspace selected:

  ```
  error: no workspace is selected
  hint: run `mlx workspace select <name>` (list them with `mlx workspace list`)
  ```

- Unknown workspace name on `select`:

  ```
  error: workspace 'researhc' does not exist or is not accessible
  hint: did you mean 'research'? List accessible workspaces with `mlx workspace list`
  ```

- Malformed name on `select`:

  ```
  error: workspace name must be kebab-case (lowercase letters, digits, hyphens)
  hint: try 'research-dev'
  ```

## Interactive Behavior

- **Prompts:** none; every command is non-interactive.
- **Destructive actions:** none; changing the selection alters only local profile state and is not confirmed.
- **Dry run:** not applicable.

## Example Sessions

### Happy path: list, select, then operate

```console
$ mlx workspace list
NAME          DESCRIPTION
research      Research team workspace
sandbox       Experiments and scratch work
$ mlx workspace select research
workspace set to 'research'
$ mlx workspace get
research
$ mlx job submit train.yaml
job 7f3a submitted in workspace research
```

### Error path: operation without a selection

```console
$ mlx job submit train.yaml
error: no workspace is selected
hint: run `mlx workspace select <name>` (list them with `mlx workspace list`)
$ echo $?
1
```

### Error path: typo on select

```console
$ mlx workspace select researhc
error: workspace 'researhc' does not exist or is not accessible
hint: did you mean 'research'? List accessible workspaces with `mlx workspace list`
$ echo $?
3
```

### Piping example

```console
$ mlx workspace list --json | jq -r '.[].name'
research
sandbox
```

## Requirements Traceability

| Requirement | Command(s) / Option(s) | Notes |
|---|---|---|
| FR-1 | `mlx workspace list` | Table of name and description from the server roster |
| FR-2 | `mlx workspace select <name>` | Persists the `workspace` key in the active profile |
| FR-3 | `mlx workspace get` | Local read of the stored selection; exit 0 with a clear statement when none |
| FR-4 | stored selection + error contract | All operation commands consume the stored selection; the no-selection error path and message are defined here |
| FR-5 | `mlx workspace select <name>` | Unknown name exits 3 with closest-match suggestion |
| FR-6 | `--json` on `list`, `select`, `get` | Shapes defined under Output Behavior |
| NFR-1 | help text + error format | Every command has help; every error carries cause and hint |
| NFR-2 | retry policy on `list`/`select` | Inherited from the FEAT-p1 client core; one informational retry line |
| NFR-3 | `mlx workspace get` | Local-only read; table rendering budget |
| NFR-4 | (toolchain gate) | Not a command; verified repository-wide |

## Out of Scope

- Workspace membership management (`member list|add|remove`, FEAT-p1 FR-24): separate surface, not designed here.
- Creating, renaming, or deleting workspaces: server-owned administration, not exposed by this group in v1.
- A `--workspace` global override flag: the persisted selection is the only mechanism, per FEAT-p1 FR-3; reconsider only if user demand appears.

## Open Questions

1. Should a default workspace be auto-selected when the user belongs to exactly one? (Carried from requirements; v1 behavior is explicit selection only.)

# CLI conventions

The `mlx` CLI follows a few conventions everywhere.
Once you know them, every command group behaves the way you expect.

## Command structure

Commands are noun-first groups with verb-first subcommands:

```text
mlx <group> <verb> [arguments] [options]
```

For example `mlx dataset list`, `mlx job submit`, `mlx deploy update`.
Command and option names are kebab-case, as are user-visible resource names (workspaces, datasets, models, ...).

## Global options

| Option | Effect |
|---|---|
| `--profile <name>` | Use a named configuration profile |
| `--workspace <name>` | Run against this workspace, overriding the persisted selection for this invocation only |
| `--output <format>` | Output format for list and get commands: `json`, `yaml`, `table` (default), `name` |
| `--json` | Shorthand for `--output json` on any command |
| `--log-level <level>` | Logging verbosity: `debug`, `info`, `warn` (default), `error`; logging goes to stderr |
| `--verbose` | Shorthand for `--log-level debug` |
| `--version` | Print the CLI version and exit |
| `--help` | Help for any command |

## Output streams

stdout carries results only.
Prompts, progress bars, and diagnostics go to stderr.
That means piping and scripting never pick up noise:

```shell
mlx job list --json | jq '.[] | select(.state == "running")'
```

## Output formats

Every list command supports `--output`:

| Format | Behavior |
|---|---|
| `table` | Human-readable table (the default) |
| `json` | One valid JSON document on stdout, nothing else |
| `yaml` | The same data as YAML |
| `name` | Exactly one element name per line, for shell piping |

`--json` is shorthand for `--output json` and works on any command.

```shell
mlx job list --output name | xargs -I{} echo "job: {}"
mlx job list --json | jq '.[] | select(.state == "running")'
```

One exception: `--json` combined with a `--follow` log stream is a usage error, since an open-ended stream cannot produce exactly one document.

## Errors

Every failure prints two lines:

```text
error: dataset 'raw-click' not found
hint: run 'mlx dataset list' to see available datasets
```

The `error` line names what went wrong, and the `hint` line names a concrete fix.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Usage error (bad arguments, invalid spec, missing selection) |
| 2 | Authentication failure |
| 3 | Server error (unreachable, rejected, internal) |
| 4 | Cancelled by user |

Scripts should branch on these rather than parse error text.

## Profiles

A profile bundles the server URL, credentials, and selected workspace under a name, stored in the platform config directory.
Switch profiles to talk to a different environment (for example staging and production):

```shell
mlx --profile prod workspace list
```

The default profile is used when `--profile` is omitted.

## Authentication state

`mlx auth login` stores your token; `mlx auth status` reports who you are, against which server; `mlx auth logout` clears the token but keeps the profile's server URL.
See [`mlx auth`](../reference/cli/auth.md).

## Completion

Shell completions are generated from the command tree, so they never drift from the CLI itself.
See [Installation](../installation.md#shell-completion).

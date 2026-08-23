# mlx secret

Workspace secrets: credentials referenced by jobs, with values that are never displayed.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `set` | `<name> (--from-literal <value> \| --from-file <path>)` | Store a credential |
| `list` | | Secret names only |
| `delete` | `<name>` | Remove the secret |

## `secret set`

Exactly one source flag is required; passing both or neither is a usage error.

| Input | Rules |
|---|---|
| `--from-literal <value>` | Value on the command line; note the shell-history risk |
| `--from-file <path>` | Value is the file's bytes; the path must exist and be readable |

Names are kebab-case and unique in the workspace.

## Value handling

Values never appear in any output, log, or error, including verbose mode.
Jobs reference secrets by name in their spec; the values are injected at run time.

## Deletion

Deletion is immediate.
Jobs submitted afterwards that reference the deleted name fail with an error naming the missing secret.

## Related

- [Secrets](../../concepts/secrets.md)
- [Job spec](../specs/job.md)

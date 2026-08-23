# mlx workspace

Workspaces and the active workspace selection.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<name>` | Create a workspace; you become its first admin |
| `list` | | Workspaces you can access |
| `select` | `<name>` | Persist the active workspace in the profile |
| `get` | | Print the active workspace |
| `delete` | `<name> [--force]` | Delete; refuses while resources remain unless `--force` |

`select` requires a name from `workspace list`; an unknown name fails server-side with the closest match suggested.
Running an operation with no workspace selected exits 1 with a hint naming `mlx workspace select`.

## `workspace member`

Membership of the active workspace; requires admin rights to modify.

| Subcommand | Arguments | Purpose |
|---|---|---|
| `list` | | Members with roles |
| `add` | `<user> [--role <role>]` | Add a user (default role: member) |
| `remove` | `<user>` | Remove a user |

## Related

- [Workspaces](../../concepts/workspaces.md)

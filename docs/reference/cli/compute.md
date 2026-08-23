# mlx compute

Machine types available to the workspace.

The authoritative set of type names comes from the server.
Job specs request one of these types in their `compute` block.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `list` | | Machine types with GPU model and count, memory, remaining quota |
| `get` | `<type>` | One type's resources and quota in detail |

An unknown type fails server-side with the closest match suggested.

## Output

- Human mode: a table for `list`, a key-value block for `get`.
- JSON mode: an array of type objects for `list`, a single object for `get`.

## Related

- [Compute and quotas](../../concepts/compute.md)
- [`mlx quota`](quota.md)

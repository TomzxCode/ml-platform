# mlx quota

Resource limits per scope, with utilization.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<scope> --resource <resource> --limit <amount>` | Set a quota, e.g. `mlx quota create workspace/research --resource gpu --limit 8` |
| `list` | `[--scope <scope>]` | Quotas with used vs. total, optionally filtered |
| `get` | `<resource> [--scope <scope>]` | One resource's limit and utilization in detail |
| `delete` | `<scope> --resource <resource>` | Remove the limit; usage falls back to platform defaults |

## Scopes

A scope is a workspace or a team, written as `workspace/<name>` or `team/<name>`.

## Resources

Resources are the quota-bearing quantities: compute, GPU, storage.

Managing quotas requires admin permission on the scope.
Jobs draw from the workspace quota at placement time; an exhausted quota surfaces as a queued job or a quota error naming the resource.

## Related

- [Compute and quotas](../../concepts/compute.md)

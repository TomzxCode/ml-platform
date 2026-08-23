# mlx infra

Clusters and machines: the infrastructure layer, for platform administrators.

## `infra cluster`

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<spec-file>` | Provision a cluster from a [spec](../specs/cluster.md); asynchronous |
| `list` | | Clusters with region and health |
| `get` | `<name>` | One cluster's details (version, capacity, capabilities) |
| `update` | `<name> [--add-machine-type <type>] [--remove-machine-type <type>] [--autoscaler-max <n>]` | Modify machine type offerings and the autoscaler ceiling |

`cluster create` returns once the cluster is registered; provisioning continues server-side until the cluster becomes healthy.

## `infra machine`

| Subcommand | Arguments | Purpose |
|---|---|---|
| `list` | `[--cluster <name>]` | Machines across clusters (or one) with state and allocation |
| `get` | `<id>` | One machine's type, cluster, state, allocation, health |

## Related

- [Clusters and machines](../../concepts/clusters.md)
- [Cluster spec](../specs/cluster.md)

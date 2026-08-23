# mlx volume

Persistent volumes: workspace storage for staging and checkpoints.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<name> --size <size> [--storage-class <class>] [--cluster <name>]` | Provision a volume |
| `list` | | Volumes with size and state |
| `get` | `<name>` | Size, storage class, cluster, mount point, state |
| `delete` | `<name>` | Release the volume; data is not recoverable |

Creating a volume draws from the workspace's storage quota; an exhausted quota fails with an error naming the resource.

## Related

- [Volumes](../../concepts/volumes.md)
- [Move data](../../guides/move-data.md)

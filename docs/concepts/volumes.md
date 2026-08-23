# Volumes

A volume is persistent storage scoped to a workspace.
Jobs use volumes for staging data and writing checkpoints that should outlive a single job.

Volumes complement [datasets](datasets.md): datasets point at cataloged data wherever it lives, while volumes are scratch and working storage owned by the workspace.

## Creating a volume

```shell
mlx volume create training-cache --size 500Gi --cluster eu-west-1
```

| Option | Effect |
|---|---|
| `--size <size>` | Capacity, for example `500Gi` |
| `--storage-class <class>` | Storage class; platform default when omitted |
| `--cluster <name>` | Cluster to place the volume in |

## Managing volumes

```shell
mlx volume list
mlx volume get training-cache
mlx volume delete training-cache
```

`volume get` shows size, storage class, cluster, mount point, and state.
Deletion is final: the data on the volume is not recoverable.

## Quota

Volume capacity draws from the workspace's storage [quota](compute.md).
Creating a volume with the quota exhausted fails with an error naming the exhausted resource.

## Commands

The full command surface is in the [`mlx volume` reference](../reference/cli/volume.md).

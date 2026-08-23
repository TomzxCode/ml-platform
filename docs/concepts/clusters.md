# Clusters and machines

Clusters are the Kubernetes clusters the platform runs on, each with a cloud, a region, and a set of machine types.
Machines are the nodes inside them, with a state and an allocation.

As a platform user you rarely need this level; it exists for platform administrators and for understanding where your work physically runs.

## Clusters

```shell
mlx infra cluster list
mlx infra cluster get eu-west-1
```

Cluster listings show region and health.
A cluster spec defines cloud, region, machine types, and size; creating a cluster from one is asynchronous: the command returns once the cluster is registered, and provisioning continues server-side until the cluster becomes healthy.

```shell
mlx infra cluster create cluster-spec.yaml
```

An existing cluster's machine type offerings and autoscaler ceiling can be modified:

```shell
mlx infra cluster update eu-west-1 --add-machine-type gpu-h100-80g --autoscaler-max 64
```

## Machines

```shell
mlx infra machine list
mlx infra machine list --cluster eu-west-1
mlx infra machine get <machine-id>
```

Machine listings show state and allocation across clusters (or one cluster), which answers "is the cluster full?" at a glance.

## Commands

The full command surface is in the [`mlx infra` reference](../reference/cli/infra.md).
The cluster spec is documented in the [specs reference](../reference/specs/cluster.md).

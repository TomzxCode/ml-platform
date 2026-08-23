# Workspaces

A workspace is the unit of isolation for your work.
Every resource you create (datasets, jobs, experiments, models, deployments, secrets, volumes, transfers) belongs to exactly one workspace.

Workspaces are similar in spirit to Kubernetes namespaces: same team, same resources, same access policy, separate from every other workspace.

## What lives in a workspace

| Resource | Created by |
|---|---|
| Datasets | `mlx dataset create` |
| Experiments | `mlx experiment create` |
| Jobs (all three types) | `mlx job submit`, `mlx batch submit` |
| Models | `mlx model register`, or a training job |
| Deployments | `mlx deploy create` |
| Secrets | `mlx secret set` |
| Volumes | `mlx volume create` |
| Transfers | `mlx data copy` |
| Service accounts | `mlx auth service-account create` |

## The active workspace

The CLI operates on one workspace at a time: the active workspace, persisted in your [profile](../guides/cli-conventions.md#profiles).

```shell
mlx workspace list
mlx workspace select research
mlx workspace get
```

Every subsequent command is scoped to that workspace automatically.
Running an operation with no workspace selected fails with a pointer to `mlx workspace select`.

For one-off overrides, pass `--workspace <name>` to any command: it scopes that single invocation without changing the persisted selection.

## Membership and roles

Roles are `admin` or `member`; the workspace's creator becomes its first admin.
Admins add and remove members:

```shell
mlx workspace member list
mlx workspace member add alice@example.com --role member
mlx workspace member remove bob@example.com
```

Non-admin members can use the workspace but cannot change its membership or quotas.

## Quotas

Each workspace holds a quota of resources (compute, GPUs, storage) that its jobs draw from.
See [Compute and quotas](compute.md).

## Deleting a workspace

```shell
mlx workspace delete research
```

Deletion is refused while the workspace still contains resources, unless `--force` is passed.
The error names the resources that block it.

## Commands

The full command surface is in the [`mlx workspace` reference](../reference/cli/workspace.md).

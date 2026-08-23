# Compute and quotas

## Compute types

A compute type is a machine type the platform offers, with GPUs, memory, and CPUs, available to your workspace under quota.
Jobs request a compute type in their spec:

```yaml
compute:
  type: gpu-a100-40g
  count: 2
```

Discover what your workspace can use:

```shell
mlx compute list
mlx compute get gpu-a100-40g
```

`compute list` shows each type's GPU model and count, memory, and remaining quota.
`compute get` shows one type's details.

## Quotas

A quota is a limit on a resource (compute, GPUs, storage) set on a scope (a workspace or a team).
Jobs in the workspace draw from that quota; when it is exhausted, submissions fail with a quota error naming the exhausted resource, and queued jobs wait for capacity.

Workspace administrators manage quotas with:

```shell
mlx quota create workspace/research --resource gpu --limit 8
mlx quota list --scope workspace/research
mlx quota get gpu --scope workspace/research
mlx quota delete workspace/research --resource gpu
```

`quota list` shows used versus total per resource, so "can I submit a 2-GPU job?" is a one-command question.

## How placement uses quota

When a job is submitted, the server places it against the workspace's quota:

1. Quota available: the job is dispatched and starts running.
2. Quota in use but not exhausted: the job may still queue briefly until compute frees up.
3. Quota exhausted: the job waits in `queued` (or the submission is rejected, depending on the resource).

The CLI never makes placement decisions; it only reports state.

## Commands

The full command surface is in the [`mlx compute`](../reference/cli/compute.md) and [`mlx quota`](../reference/cli/quota.md) references.

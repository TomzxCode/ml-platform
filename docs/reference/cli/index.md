# CLI reference

The full `mlx` command tree.
Every command accepts the [global options](../../guides/cli-conventions.md#global-options).

```text
mlx auth login|logout|status
mlx auth api-key create|list|get|delete
mlx auth service-account create|list|get|delete
mlx batch submit|list|get|logs|cancel
mlx completion bash|zsh|fish
mlx compute list|get
mlx data copy
mlx dataset create|list|get|delete
mlx deploy create|list|get|update|stop
mlx experiment create|list|get
mlx infra cluster create|list|get|update
mlx infra machine list|get
mlx job submit|list|get|logs|cancel
mlx model register|list|get
mlx quota create|list|get|delete
mlx secret set|list|delete
mlx user list|get
mlx volume create|list|get|delete
mlx workspace create|list|get|select|delete
mlx workspace member list|add|remove
```

| Group | Covers |
|---|---|
| [`auth`](auth.md) | Login state, API keys, service accounts |
| [`batch`](batch.md) | Batch inference fast path |
| [`completion`](completion.md) | Shell completion scripts |
| [`compute`](compute.md) | Machine types available to the workspace |
| [`data`](data.md) | Data copies between locations |
| [`dataset`](dataset.md) | Dataset catalog entries |
| [`deploy`](deploy.md) | Online inference deployments |
| [`experiment`](experiment.md) | Experiments and their runs |
| [`infra`](infra.md) | Clusters and machines |
| [`job`](job.md) | All job types and their lifecycle |
| [`model`](model.md) | Model registry |
| [`quota`](quota.md) | Resource limits per scope |
| [`secret`](secret.md) | Workspace secrets |
| [`user`](user.md) | Users and memberships |
| [`volume`](volume.md) | Persistent volumes |
| [`workspace`](workspace.md) | Workspaces, the active selection, and members |

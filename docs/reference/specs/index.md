# Specs

Several CLI commands take a small YAML file describing the resource.
This section defines each one.

| Spec | Consumed by |
|---|---|
| [Job spec](job.md) | `mlx job submit`, `mlx batch submit` |
| [Dataset spec](dataset.md) | `mlx dataset create` |
| [Experiment spec](experiment.md) | `mlx experiment create` |
| [Cluster spec](cluster.md) | `mlx infra cluster create` |

## Shared rules

- Specs are validated locally before any server call; the first problem found is named in the error.
- Names are kebab-case and unique within the workspace.
- Secret values never appear in specs; jobs reference secrets by name and the values are injected at run time.

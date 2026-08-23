# Jobs

A job is a unit of work that the platform executes on compute.
There are three job types, and they share one spec format, one lifecycle, and one set of lifecycle commands.

## Job types

| Type | What it does | Key inputs |
|---|---|---|
| `processing` | Transforms input datasets into new output datasets | Container image, inputs, outputs |
| `training` | Trains a model on a dataset | Container image, dataset, compute, optional experiment and model name |
| `batch-inference` | Runs a registered model over a dataset | Model `name:version`, inputs, output dataset |

All three are submitted through the same [job spec](../reference/specs/job.md), discriminated by the `type` field:

```shell
mlx job submit train-ctr.yaml
```

Batch inference also has a dedicated fast path with its own lifecycle commands:

```shell
mlx batch submit score-nightly.yaml
```

## Lifecycle

Every job, regardless of type, moves through the same states:

```text
queued -> running -> succeeded
                     -> failed
                     -> cancelled
```

- `queued`: accepted and persisted; waiting for quota and compute.
- `running`: dispatched to Kubernetes; logs are streaming.
- `succeeded`: finished; outputs (datasets, model versions) are registered.
- `failed`: finished with an error; inspect logs for the cause.
- `cancelled`: stopped at user request via `job cancel`.

States a newer server might add render verbatim instead of failing the CLI.
Submission is retry-safe: each invocation carries an idempotency key, so a network retry never creates a duplicate job.

## Placement

The server decides where a job runs, against the workspace's compute quota and the compute type requested in the spec.
If the quota is exhausted, the job stays `queued` until capacity frees up.
See [Compute and quotas](compute.md).

## Tracking jobs

```shell
mlx job list                       # all jobs in the workspace
mlx job list --type training       # filter by type
mlx job list --state running       # filter by state
mlx job get <job-id>               # one job's current state
mlx job logs <job-id>              # print logs
mlx job logs <job-id> --follow     # keep streaming
mlx job cancel <job-id>            # request cancellation
```

The `batch` group mirrors these commands for batch inference jobs only.

## Secrets and environment

Jobs can reference workspace [secrets](secrets.md) by name and declare environment variables, both in the spec.
Secret values are injected at run time and never appear in specs, logs, or CLI output.

## Commands

The full command surface is in the [`mlx job`](../reference/cli/job.md) and [`mlx batch`](../reference/cli/batch.md) references.

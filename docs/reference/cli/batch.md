# mlx batch

The batch inference fast path: submit batch inference specs and run the job lifecycle for batch jobs only.

`batch` and `job` share the same submission machinery and job records.
`batch list` output equals `job list --type batch-inference`, and `batch get/logs/cancel` accept the same job ids as their `job` counterparts.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `submit` | `<spec-file>` | Submit a batch inference job; prints the job id |
| `list` | `[--state <state>]` | List batch inference jobs |
| `get` | `<id>` | Print one batch job's current state |
| `logs` | `<id> [--follow]` | Print the job's logs; `--follow` keeps the stream open |
| `cancel` | `<id>` | Request cancellation |

## Spec

`batch submit` takes the same YAML document as `job submit`, with `type: batch-inference`:

```yaml
name: nightly-ctr-scoring
type: batch-inference
compute:
  type: gpu-a100-40g
model: ctr-estimator:3
inputs:
  - dataset: cleaned-clicks
output:
  dataset: nightly-scores
```

The full field reference is in the [job spec](../specs/job.md).

`batch submit` rejects a spec whose `type` is not `batch-inference` locally, before any server call, naming the actual type and suggesting `job submit`.

## Behavior notes

- `batch get` shows the output dataset name and location once the job has registered them.
- `batch list` follows pagination to exhaustion, matching `job list --type batch-inference` exactly.
- Logs of an already-finished job print the stored lines and exit 0, with or without `--follow`.
- `--json` with `--follow` is a usage error, matching `job logs`.
- Cancelling an already-terminal job exits 1 naming the job's current state.

## Related

- [Jobs](../../concepts/jobs.md)
- [Score a dataset](../../guides/score-a-dataset.md)
- [`mlx job`](job.md)

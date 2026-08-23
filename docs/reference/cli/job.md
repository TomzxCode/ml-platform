# mlx job

All job types (data processing, training, batch inference) and their lifecycle.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `submit` | `<spec-file>` | Submit a job declared by a [spec](../specs/job.md); prints the job id |
| `list` | `[--type <type>] [--state <state>]` | Jobs with id, type, state |
| `get` | `<id>` | One job's current state |
| `logs` | `<id> [--follow]` | Print logs; `--follow` keeps the stream open |

Streaming notes:

- `--follow` ends by itself when the job reaches a terminal state; interrupting it with Ctrl+C also exits 0 and never affects the running job.
- `--json` cannot be combined with `--follow` (an open stream cannot produce exactly one document); without `--follow`, JSON mode returns the accumulated lines as one document.
| `cancel` | `<id>` | Request cancellation |

## Submission

The spec's `type` field (`processing`, `training`, `batch-inference`) selects the job type.
Validation runs locally before any server call: YAML parsing, required fields per type, enum checks.
Reference existence (datasets, models, secrets, compute types) is confirmed by the server, which owns the authoritative check; unknown references are named in the error.

Submissions are safe to retry: each invocation carries an idempotency key, so a network-retry never creates a duplicate job.

## Lifecycle

All job types share one state model: `queued`, `running`, `succeeded`, `failed`, `cancelled`.
Unknown state values (from a newer server) render verbatim rather than failing.

## Listing

`job list` follows the server's pagination to exhaustion before rendering, so you always see the complete result set (in both human and JSON modes).

## Batch inference

For batch inference jobs only, the [`mlx batch`](batch.md) group offers the same lifecycle with a dedicated submit.

## Related

- [Jobs](../../concepts/jobs.md)
- [Job spec](../specs/job.md)
- [Train a model](../../guides/train-a-model.md)

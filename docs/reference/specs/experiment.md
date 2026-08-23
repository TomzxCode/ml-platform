# Experiment spec

The experiment spec creates a named grouping for training runs.
Consumed by `mlx experiment create`.

## Fields

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case, unique in the workspace |
| description | string | No | Single line, shown in listings; a value with newlines is rejected |

Unknown fields are ignored with a warning, never rejected.

## Example

```yaml
name: ctr-baseline
description: First pass on CTR estimation
```

## Attaching runs

Training jobs attach by naming the experiment in their spec's `experiment` field; see the [job spec](job.md).
The experiment must exist before the training job is submitted: an unknown name fails at submit time rather than auto-creating the experiment.

## Validation

- The file parses as YAML; the parse error is named otherwise.
- `name` is present and kebab-case.
- Uniqueness is confirmed server-side on create.

## Related

- [Experiments](../../concepts/experiments.md)
- [`mlx experiment`](../cli/experiment.md)

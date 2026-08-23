# Score a dataset

Batch inference runs a registered model version over an input dataset and registers the scored output as a new dataset.
This guide scores a dataset with a model you trained or registered earlier.

## 1. Confirm your inputs

You need:

- a registered model version (`mlx model get ctr-estimator`), and
- an input dataset in the catalog (`mlx dataset list`).

## 2. Write the batch inference spec

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

| Field | Purpose |
|---|---|
| `model` | The `name:version` to score with |
| `inputs[].dataset` | Dataset catalog names to score |
| `output.dataset` | Dataset name registered with the results on success |
| `output.location` | Optional explicit destination URI |

The full field list is in the [job spec reference](../reference/specs/job.md).

## 3. Submit

Use the batch fast path:

```shell
mlx batch submit score-nightly.yaml
```

Or the general job command with the same file; they are equivalent:

```shell
mlx job submit score-nightly.yaml
```

## 4. Track the job

```shell
mlx batch list --state running
mlx batch get <job-id>
mlx batch logs <job-id> --follow
```

## 5. Use the output

On success, the output dataset appears in the catalog:

```shell
mlx dataset get nightly-scores
```

From there it is a first-class dataset: other jobs can read it, you can copy it elsewhere with [`mlx data copy`](move-data.md), or score it again with a newer model version.

## Nightly scoring

Batch inference pairs well with an API key and a scheduler.
See [Automation](automation.md) for running this loop unattended.

# Job spec

The job spec is one YAML file covering all three job types, discriminated by the `type` field.

Consumed by `mlx job submit` (all types) and `mlx batch submit` (`type: batch-inference` only).

## Common fields

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case, shown in listings |
| type | string | Yes | `processing`, `training`, or `batch-inference` |
| compute.type | string | Yes | A type from `mlx compute list` |
| compute.count | integer | No | Defaults to 1 |
| description | string | No | Free-form, not shown in listings |
| secrets | string list | No | Names from `mlx secret list`; values never appear |
| env | string map | No | Environment variables; scalar values (string, number, boolean) coerced to strings, nested structures rejected naming the key |

## `type: processing`

Transforms input datasets into new output datasets.

| Field | Type | Required | Constraints |
|---|---|---|---|
| inputs[].dataset | string | Yes | Dataset catalog name |
| inputs[].mount | string | No | Path the job reads the input from |
| outputs[].name | string | Yes | Dataset name registered in the catalog on success |
| outputs[].location | string | No | Explicit destination URI; generated when omitted |
| code.image | string | Yes | Container image to run |
| code.entrypoint | string | No | Overrides the image entrypoint |
| code.args | string list | No | Arguments passed to the entrypoint |

## `type: training`

Trains a model on one dataset.

| Field | Type | Required | Constraints |
|---|---|---|---|
| code.image | string | Yes | Container image to run |
| code.entrypoint | string | No | Overrides the image entrypoint |
| code.args | string list | No | Arguments passed to the entrypoint |
| dataset | string | Yes | Dataset catalog name trained on |
| experiment | string | No | Experiment name the run attaches to; must already exist |
| model.name | string | No | Registers the trained artifact under this name |
| hyperparameters | map | No | Scalars and nested maps of scalars passed to the training code; arrays rejected naming the key |

On success with `model.name` set, the final checkpoint is registered as a new version of that model, with lineage to this job.

## `type: batch-inference`

Runs a registered model over input datasets and registers the scored output.

| Field | Type | Required | Constraints |
|---|---|---|---|
| model | string | Yes | `name:version` of a registered model (kebab-case name, positive integer version) |
| inputs[].dataset | string | Yes | Dataset catalog name scored by the model |
| output.dataset | string | Yes | Output dataset name registered on success |
| output.location | string | No | Explicit destination URI |

## Example: training

```yaml
name: ctr-baseline-run
type: training
compute:
  type: gpu-a100-40g
  count: 2
code:
  image: ghcr.io/acme/ctr-trainer:1.4
  entrypoint: python train.py
dataset: cleaned-clicks
experiment: ctr-baseline
model:
  name: ctr-estimator
hyperparameters:
  lr: 0.001
  epochs: 10
secrets: [wandb-key]
```

## Example: batch inference

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

## Validation

Local checks run before any server call; each failure exits 1 and names the offending field:

- The file parses as YAML.
- Required fields per type are present, including the fields selected by `type`.
- Enum values are valid (`type`, `compute.type` syntax).

References (datasets, models, secrets, compute types) are structurally valid locally, then confirmed against the server, which owns the authoritative check.
Unknown references are named in the error.

Unknown fields in the spec file are ignored with a warning, never rejected, so newer spec versions do not break older CLI versions.

## Related

- [Jobs](../../concepts/jobs.md)
- [`mlx job`](../cli/job.md), [`mlx batch`](../cli/batch.md)

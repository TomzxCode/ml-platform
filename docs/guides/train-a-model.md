# Train a model

This guide walks through a full training workflow: prepare data, submit training, track the run, and end with a registered model version.

## 1. Prepare the dataset

Register your training data in the [catalog](../concepts/datasets.md) if you have not already:

```shell
cat > raw-clicks.yaml <<'EOF'
name: raw-clicks
location: s3://ml-bucket/raw/clicks
format: parquet
description: Raw click events from the collector
labels: [daily, clicks]
EOF

mlx dataset create raw-clicks.yaml
```

If the data is not in storage yet, move it there first with [`mlx data copy`](move-data.md).

## 2. Create an experiment

Group the runs you are about to launch under an [experiment](../concepts/experiments.md).
Create it before submitting: a training spec that names an unknown experiment is rejected at submit time.

```shell
cat > ctr-baseline.yaml <<'EOF'
name: ctr-baseline
description: First pass on CTR estimation
EOF

mlx experiment create ctr-baseline.yaml
```

## 3. Add the secrets your code needs

If the training code talks to external systems, store those credentials as [secrets](../concepts/secrets.md):

```shell
mlx secret set wandb-key --from-file ./wandb.token
```

## 4. Write the training spec

A training job runs a container image of your choosing on the compute you request:

```yaml
name: ctr-baseline-run
type: training
compute:
  type: gpu-a100-40g
  count: 2
code:
  image: ghcr.io/acme/ctr-trainer:1.4
  entrypoint: python train.py
dataset: raw-clicks
experiment: ctr-baseline
model:
  name: ctr-estimator
hyperparameters:
  lr: 0.001
  epochs: 10
secrets: [wandb-key]
```

The key fields:

| Field | Purpose |
|---|---|
| `compute.type` / `compute.count` | Machine type and how many |
| `code.image` | The container image that runs |
| `dataset` | The cataloged dataset to train on |
| `experiment` | The experiment this run attaches to |
| `model.name` | Registers the final checkpoint under this name on success |
| `hyperparameters` | Free-form map passed through to your code |

The full field list is in the [job spec reference](../reference/specs/job.md).

## 5. Submit and watch

```shell
mlx job submit train-ctr.yaml
mlx job logs <job-id> --follow
```

The job moves through `queued` and `running`.
Press Ctrl+C to stop following; the job keeps running.

## 6. Check the outcome

```shell
mlx job get <job-id>
```

On success, the final checkpoint is registered as a new version of `ctr-estimator`:

```shell
mlx model get ctr-estimator
```

The version records its artifact location and its lineage to this job.

## 7. Iterate

Submit variations with different names and hyperparameters; all runs that set `experiment: ctr-baseline` collect under it:

```shell
mlx experiment get ctr-baseline
```

Compare run metrics there and keep the best model version.

## Next steps

- [Deploy](deploy-a-model.md) the winning version for online inference.
- [Score](score-a-dataset.md) a dataset with it in batch.

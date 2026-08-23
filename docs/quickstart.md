# Quickstart

This walkthrough takes you from a fresh install to a trained, registered, and served model.
Everything happens through the `mlx` CLI.

You will:

1. Log in and select a workspace.
2. Register a dataset.
3. Train a model on it.
4. Find the model the training job registered.
5. Deploy it for online inference.
6. Score a dataset in batch with it.
7. Stop the deployment.

## 1. Log in

Point the CLI at your API server and authenticate with your token:

```shell
mlx auth login --server https://ml.internal.example.com
```

The CLI prompts for the token without echoing it.
Confirm the session:

```shell
mlx auth status
```

## 2. Select a workspace

All resources live in a [workspace](concepts/workspaces.md).

```shell
mlx workspace list
mlx workspace select research
```

## 3. Register a dataset

Describe your dataset in a small YAML file, `raw-clicks.yaml`:

```yaml
name: raw-clicks
location: s3://ml-bucket/raw/clicks
format: parquet
description: Raw click events from the collector
labels: [daily, clicks]
```

Register it in the catalog:

```shell
mlx dataset create raw-clicks.yaml
```

See the [dataset spec](reference/specs/dataset.md) for all fields.

## 4. Train a model

Describe the training job in `train-ctr.yaml`:

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

Submit it and watch the logs:

```shell
mlx job submit train-ctr.yaml
mlx job logs <job-id> --follow
```

When the job succeeds, its final checkpoint is registered as version 1 of `ctr-estimator` in the [model registry](concepts/models.md), with lineage back to the job.
See the [job spec](reference/specs/job.md) for every field, and the [training guide](guides/train-a-model.md) for the full workflow.

## 5. Check the registered model

```shell
mlx model list
mlx model get ctr-estimator
```

`model get` shows each version with its artifact location and the job that produced it.

## 6. Deploy for online inference

Serve the trained model behind an endpoint:

```shell
mlx deploy create ctr-estimator:1 --endpoint-name ctr-api --min-replicas 2 --max-replicas 8
```

The command prints the endpoint reference.
The endpoint name also names the deployment; check its state with:

```shell
mlx deploy get ctr-api
```

It shows `creating` right after the request and `serving` once the endpoint is ready.
See [Deploy a model](guides/deploy-a-model.md) for scaling, rollouts, and stopping.

## 7. Score a dataset in batch

Run the same model over a whole dataset, `score-nightly.yaml`:

```yaml
name: nightly-ctr-scoring
type: batch-inference
compute:
  type: gpu-a100-40g
model: ctr-estimator:1
inputs:
  - dataset: raw-clicks
output:
  dataset: nightly-scores
```

Submit and track it with the batch fast path:

```shell
mlx batch submit score-nightly.yaml
mlx batch get <job-id>
```

On success, `nightly-scores` appears in the dataset catalog.
See [Score a dataset](guides/score-a-dataset.md).

## 8. Clean up

Stop serving the model when you are done:

```shell
mlx deploy stop ctr-api
```

## Where to go next

- [Concepts](concepts/index.md) for how the pieces fit together.
- [CLI reference](reference/cli/index.md) for the full command surface.
- [Troubleshooting](guides/troubleshooting.md) when something does not behave.

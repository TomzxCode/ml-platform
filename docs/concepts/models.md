# Models

A model is a registered family of versions.
Each version points at an artifact (the trained checkpoint) and records lineage: which job produced it.

The registry is what connects training to serving: only registered model versions can be deployed or used for batch inference.

## Where versions come from

There are two ways a model version appears in the registry:

| Source | How |
|---|---|
| A training job | The spec's `model.name` field registers the final checkpoint on success |
| External artifact | `mlx model register` records an artifact you trained elsewhere |

Registering an external artifact:

```shell
mlx model register s3://ml-bucket/models/ctr-v3.bin --name ctr-estimator
```

## Versions

Versions are positive integers starting at 1, newest first in listings.
When `--version` is omitted, the server assigns the next version under the name.
An explicit version that already exists is rejected as a duplicate.
Registering under a brand-new name creates the model.

One caveat on local paths: `mlx model register ./model.pt --name my-model` records the artifact's location only (normalized to a `file://` URI); no bytes are uploaded.
Jobs running on other machines cannot read a local path, so prefer a storage URI (`s3://`, `gs://`) for artifacts jobs will use.

## Referencing a model version

Deployments and batch inference jobs reference a model as `name:version`:

```text
ctr-estimator:3
```

## Lineage

`mlx model get ctr-estimator` shows every version with:

- its artifact location,
- its creation time, and
- the job that produced it, when the version came from a training job.

Versions registered externally carry no lineage, and the lineage line is simply omitted for them.
Lineage makes "which run produced the model we are serving?" a one-command question.

## Commands

The full command surface is in the [`mlx model` reference](../reference/cli/model.md).

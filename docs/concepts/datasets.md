# Datasets

A dataset is a cataloged data source that jobs can reference by name.
The catalog entry points at data that already exists in storage; it does not move or copy the data.

Datasets make jobs reproducible and auditable: a training spec says `dataset: raw-clicks`, not a hard-coded bucket path.

## Anatomy

| Field | Example |
|---|---|
| name | `raw-clicks` (kebab-case, unique in the workspace) |
| location | `s3://ml-bucket/raw/clicks`, `gs://...`, `file://...` |
| format | `parquet`, `csv`, `jsonl` |
| description | One line shown in listings |
| labels | Free-form tags, for example `[daily, clicks]` |

Datasets are created from a small [spec file](../reference/specs/dataset.md):

```shell
mlx dataset create raw-clicks.yaml
mlx dataset list
mlx dataset get raw-clicks
```

## How datasets relate to jobs

- Data processing jobs read input datasets and register new datasets as outputs.
- Training jobs read one dataset.
- Batch inference jobs read input datasets and register their scored output as a dataset.

The output dataset appears in the catalog only when the job succeeds, so a cataloged dataset is a green light: its producing job finished.

## Deleting a dataset

```shell
mlx dataset delete raw-clicks
```

Deletion removes the catalog entry only, never the underlying data.
Jobs that later reference the deleted name fail with an error naming the missing dataset.

## Moving data into place

To copy data between locations before cataloging it, use [data transfers](transfers.md).
For scratch space during staging, see [volumes](volumes.md).

## Commands

The full command surface is in the [`mlx dataset` reference](../reference/cli/dataset.md).

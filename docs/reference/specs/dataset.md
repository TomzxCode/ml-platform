# Dataset spec

The dataset spec registers a catalog entry pointing at existing data.
Consumed by `mlx dataset create`.

## Fields

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case, unique in the workspace |
| location | string | Yes | Storage URI, for example `s3://`, `gs://`, or `file://` |
| format | string | Yes | One of the catalog's known formats (parquet, csv, jsonl) |
| description | string | No | One line shown in listings |
| labels | string list | No | Free-form tags for filtering |

## Example

```yaml
name: raw-clicks
location: s3://ml-bucket/raw/clicks
format: parquet
description: Raw click events from the collector
labels: [daily, clicks]
```

## Validation

- The file parses as YAML; the parse error is named otherwise.
- Every required field is present; the first missing field is named.
- `format` is a known format; accepted values are listed on error.
- `location` uses a recognized URI scheme.
- `name` is kebab-case; uniqueness is confirmed server-side.

## Related

- [Datasets](../../concepts/datasets.md)
- [`mlx dataset`](../cli/dataset.md)

# Move data

This guide covers getting data where it needs to be: server-executed transfers between locations, and volumes for staging and checkpoints.

## Copy between locations

`mlx data copy` asks the server to copy data from a source to a destination:

```shell
mlx data copy s3://ml-bucket/raw/clicks gs://eu-data/clicks
```

Both arguments are storage URIs (`s3://`, `gs://`, `file://`, ...).
Access is checked before the transfer starts, so permission problems fail fast.

Progress (bytes copied, rate) streams to stderr, and a summary prints on success.

## Resume an interrupted copy

If a copy is interrupted, rerun it with `--resume`:

```shell
mlx data copy s3://ml-bucket/raw/clicks gs://eu-data/clicks --resume
```

The transfer continues from where it stopped, without re-copying completed parts.
With no previous state for the pair, the command behaves like a fresh copy and says so.

## Script transfers

With `--json` the command prints one JSON summary document, which pairs well with a scheduler:

```shell
mlx --json data copy s3://ml-bucket/raw/clicks gs://eu-data/clicks | jq .bytes
```

## Stage data on a volume

When jobs need working storage that outlives a single run (caches, checkpoints), create a [volume](../concepts/volumes.md):

```shell
mlx volume create training-cache --size 500Gi
mlx volume get training-cache
```

The `volume get` output shows the mount point jobs use.

## Typical flow

1. Copy raw data into storage: `mlx data copy`.
2. Register it in the catalog: `mlx dataset create`.
3. Optionally stage it on a volume for faster job starts: `mlx volume create`.
4. Submit jobs that reference the dataset by name.

See [Datasets](../concepts/datasets.md) for the catalog side of this flow.

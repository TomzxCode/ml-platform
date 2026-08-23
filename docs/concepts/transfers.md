# Data transfers

A transfer is a server-executed copy of data from a source location to a destination location.
Use it to move data into the storage your jobs read from, or to export results elsewhere.

```shell
mlx data copy s3://ml-bucket/raw/clicks gs://eu-data/clicks
```

The server executes the copy; your laptop can go offline once the request is accepted.

## Source and destination

Both arguments are storage URIs with recognized schemes, for example `s3://`, `gs://`, and `file://`.
Read access on the source and write access on the destination are checked before any byte moves, so a permissions problem fails fast instead of mid-transfer.

## Progress

Human mode prints progress (bytes copied, rate) to stderr and a summary on success.
With `--json`, the command prints a single JSON summary document suitable for scripting.

## Resume

A copy that is interrupted leaves enough state to continue:

```shell
mlx data copy s3://ml-bucket/raw/clicks gs://eu-data/clicks --resume
```

`--resume` continues from where the previous attempt stopped, without re-copying completed parts.
With no recorded state for the pair, it behaves like a fresh copy and says so.

## Commands

The full command surface is in the [`mlx data` reference](../reference/cli/data.md).

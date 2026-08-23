# mlx data

Server-executed copies between locations.

## `data copy`

```shell
mlx data copy <source> <destination> [--resume]
```

| Argument | Required | Rules |
|---|---|---|
| source | Yes | Storage URI readable by you (`s3://`, `gs://`, `file://`, ...) |
| destination | Yes | Storage URI writable by you |
| `--resume` | No | Continue an interrupted copy for this pair instead of restarting |

Behavior:

- Access on both sides is checked before any byte moves.
- Progress (bytes, rate) goes to stderr in human mode; a summary prints on success.
- JSON mode prints one summary document.
- With no recorded state for the pair, `--resume` behaves like a fresh copy and says so.

## Related

- [Data transfers](../../concepts/transfers.md)
- [Move data](../../guides/move-data.md)

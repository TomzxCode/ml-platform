# mlx dataset

Dataset catalog entries.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<spec-file>` | Register a dataset from a [spec](../specs/dataset.md) |
| `list` | | Datasets in the workspace |
| `get` | `<name>` | One dataset's location, format, version |
| `delete` | `<name>` | Remove the catalog entry (never the data) |

Specs are validated locally first: YAML must parse, required fields must be present, the format must be known, and the location scheme must be recognized.
Uniqueness of the name is confirmed server-side.

## Deletion semantics

`delete` removes the catalog entry only.
The underlying data is untouched.
Jobs that later reference the deleted name fail with an error naming the missing dataset.

## Related

- [Datasets](../../concepts/datasets.md)
- [Dataset spec](../specs/dataset.md)

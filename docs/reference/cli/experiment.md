# mlx experiment

Experiments: named groupings of training runs.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<spec-file>` | Create an experiment from a [spec](../specs/experiment.md) |
| `list` | | Experiments in the workspace |
| `get` | `<name>` | One experiment's runs with their metrics |

## Attaching runs

Training jobs attach to an experiment through the training spec's `experiment` field.
The experiment must already exist: submitting a training spec that names an unknown experiment fails at submit time with the experiment named.
Every training job that names the experiment becomes one of its runs; the CLI reads runs and metrics from the server's records.

## Validation

Specs are validated locally (YAML parses, `name` is kebab-case, `description` is a single line); name uniqueness is confirmed server-side.
Unknown spec fields are ignored with a warning.

## Related

- [Experiments](../../concepts/experiments.md)
- [Train a model](../../guides/train-a-model.md)

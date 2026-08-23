# mlx model

The model registry.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `register` | `<artifact-path> --name <name> [--version <v>]` | Register an externally trained artifact |
| `list` | | Registered models with version counts |
| `get` | `<name>` | Versions with artifact locations, creation times, lineage |

## `model register`

| Input | Required | Rules |
|---|---|---|
| artifact path | Yes | Bare path that exists locally, or a URI with a recognized scheme (`s3://`, `gs://`, `file://`) |
| `--name` | Yes | Registry name, kebab-case, reused across versions; a new name creates the model |
| `--version` | No | Positive integer; the next version under the name when omitted |

Validation is local first: the path must exist (for bare paths) or use a recognized scheme, the name must be kebab-case, the version must be a positive integer.
An explicit version that already exists is rejected server-side as a duplicate naming the model and version.

A bare local path is recorded as a `file://` URI; the CLI notes on stderr that jobs on other machines cannot read it.
No artifact bytes are uploaded: registration records the location only, so prefer a storage URI for artifacts that jobs or deployments will consume.

Training jobs register their final checkpoint automatically through the spec's `model.name` field; `model register` is for artifacts trained elsewhere.

## Model references

Deployments and batch inference jobs reference a model as `name:version`.

## Output

- `list`: a table of model names and version counts.
- `get`: one block per version, newest first, with artifact location, creation time, and a lineage line when the version came from a training job.
- JSON mode: an array of model objects, or one object with a `versions` array for `get`.

## Related

- [Models](../../concepts/models.md)

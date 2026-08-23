# mlx deploy

Online inference deployments.

## Subcommands

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<model>:<version> [--endpoint-name <name>] [--replicas <n>] [--min-replicas <n>] [--max-replicas <n>]` | Deploy a model version and print the endpoint reference |
| `list` | | Deployments with name, state, served version, endpoint |
| `get` | `<name>` | One deployment: served version, endpoint, state, scaling, rollout |
| `update` | `<name> [--model <model>:<version>] [--replicas <n>] [--min-replicas <n>] [--max-replicas <n>]` | Update the served model and/or scaling |
| `stop` | `<name>` | Stop serving and release compute |

## Names

The deployment is addressed by its endpoint name: the `--endpoint-name` chosen at create, or the generated one.
Create prints the endpoint reference; `get`, `update`, and `stop` take the same name.

## Model reference

Deployments reference a model as `name:version` (kebab-case name, positive integer version), which must exist in the registry.
A malformed reference exits 1 with the expected form; an unknown reference is confirmed and named by the server.

## Scaling flags

| Flag | Rules |
|---|---|
| `--replicas <n>` | Static count; positive integer; mutually exclusive with the bounds |
| `--min-replicas <n>` | Autoscale lower bound; passed together with `--max-replicas` |
| `--max-replicas <n>` | Autoscale upper bound; min must not exceed max |
| `--endpoint-name <name>` | Kebab-case; generated when omitted; must be free in the workspace |

With no scaling flags, create records a static count of 1.
Setting `--replicas` on update clears autoscaling; setting bounds replaces a static count.

## Update semantics

`deploy update` accepts any non-empty subset of the flags; an empty subset is a usage error, and omitted flags keep their values.
Updating the served model triggers a rollout without dropping the endpoint: `deploy get` reports the rollout (from version, to version, status) until it completes.

## Retry safety

`deploy create` carries an idempotency key, so a retried create (after a transient network failure) does not create a second deployment.

## Stopping

`deploy stop` exits 1 naming the deployment when it does not exist or is already stopped.

## Related

- [Deployments](../../concepts/deployments.md)
- [Deploy a model](../../guides/deploy-a-model.md)

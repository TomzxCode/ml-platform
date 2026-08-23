# Deployments

A deployment serves one registered model version for online inference behind an endpoint.
You create it from the CLI, it runs on the platform's compute, and the platform keeps it available until you stop it.

## Creating a deployment

```shell
mlx deploy create ctr-estimator:3 --endpoint-name ctr-api --min-replicas 2 --max-replicas 8
```

The command prints the endpoint reference that clients call.
The endpoint name (chosen with `--endpoint-name`, generated when omitted) is also the deployment's name: `deploy get`, `deploy update`, and `deploy stop` all address the deployment by it.

Creation returns as soon as the server accepts the request (state `creating`); the endpoint becomes ready server-side, and `deploy get` shows when the deployment is `serving`.
An endpoint name already in use in the workspace is rejected with an error naming it.

## States

A deployment moves through `creating`, `serving`, `updating` (during a model rollout), and ends `stopped` or `failed`.
Unknown states from a newer server render verbatim.

## Scaling

A deployment scales one of two ways:

| Mode | Flags | Behavior |
|---|---|---|
| Static replicas | `--replicas <n>` | A fixed replica count |
| Autoscaling | `--min-replicas <n> --max-replicas <n>` | Scales between bounds with load |

With no scaling flags, a deployment starts with a static count of 1.
The two modes are mutually exclusive.
Switching modes replaces the previous setting: passing `--replicas` clears autoscaling bounds, and passing bounds replaces a static count.
Counts must be positive integers, and min must not exceed max.

## Updating a deployment

```shell
mlx deploy update ctr-api --model ctr-estimator:4
mlx deploy update ctr-api --min-replicas 4 --max-replicas 12
```

`deploy update` accepts any non-empty subset of the flags; omitted flags keep their current values.
Updating the served model triggers a rollout: the endpoint keeps serving, `deploy get` reports the rollout's progress (from version, to version, status), and the `model` field shows the target version while the rollout is in flight.

## Stopping a deployment

```shell
mlx deploy stop ctr-api
```

Stopping releases the deployment's compute.
Stopping a deployment that does not exist or is already stopped exits 1 naming the deployment and its state.
The model stays in the registry, so you can deploy it again at any time.

## Commands

The full command surface is in the [`mlx deploy` reference](../reference/cli/deploy.md).

# Deploy a model

A deployment serves a registered model version behind an endpoint for online inference.
This guide covers creating, scaling, updating, and stopping deployments.

## 1. Pick a model version

```shell
mlx model get ctr-estimator
```

Deployments reference a version as `name:version`.

## 2. Create the deployment

```shell
mlx deploy create ctr-estimator:3
```

The command prints the endpoint reference your clients call.
Add options to shape the serving:

```shell
mlx deploy create ctr-estimator:3 \
  --endpoint-name ctr-api \
  --min-replicas 2 \
  --max-replicas 8
```

| Option | Effect |
|---|---|
| `--endpoint-name <name>` | Names the endpoint and the deployment; generated when omitted |
| `--replicas <n>` | Fixed replica count |
| `--min-replicas <n>` / `--max-replicas <n>` | Autoscaling bounds |

Static replicas and autoscaling bounds are mutually exclusive.
With no scaling flags, the deployment runs a static count of 1.

The command returns as soon as the server accepts it (state `creating`).
The endpoint becomes ready server-side; `deploy get` shows the deployment `serving` once it is.

## 3. Inspect the deployment

```shell
mlx deploy list
mlx deploy get ctr-api
```

`deploy get` shows the served version, endpoint, state, and scaling configuration.

## 4. Roll out a new version

```shell
mlx deploy update ctr-api --model ctr-estimator:4
```

The endpoint stays up: the new version rolls out behind it.
While the rollout is in flight, `deploy get` reports its progress (from version, to version, status), and the deployment's `model` field shows the target version.

## 5. Rescale

```shell
mlx deploy update ctr-api --replicas 3
mlx deploy update ctr-api --min-replicas 4 --max-replicas 12
```

Omitted flags keep their current values, and at least one flag is required.
Passing `--replicas` switches the deployment from autoscaling to a static count; passing bounds switches it back.

## 6. Stop serving

```shell
mlx deploy stop ctr-api
```

Stopping releases the deployment's compute.
The model version stays in the registry and can be redeployed at any time.

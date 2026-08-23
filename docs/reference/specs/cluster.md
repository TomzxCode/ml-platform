# Cluster spec

The cluster spec provisions a new Kubernetes cluster for the platform.
Consumed by `mlx infra cluster create`.

Creation is asynchronous: the command returns once the cluster is registered, and provisioning continues server-side until the cluster reaches a healthy state.

## Fields

| Field | Type | Required | Constraints |
|---|---|---|---|
| name | string | Yes | Kebab-case |
| cloud | string | Yes | Cloud provider |
| region | string | Yes | Cloud region |
| machine-types | string list | Yes | Machine types offered by the cluster |
| size | object | No | Capacity settings, including the autoscaling ceiling |

## Example

```yaml
name: eu-west-1
cloud: aws
region: eu-west-1
machine-types: [cpu-general-8, gpu-a100-40g]
size:
  autoscaler-max: 32
```

## Related

- [Clusters and machines](../../concepts/clusters.md)
- [`mlx infra`](../cli/infra.md)

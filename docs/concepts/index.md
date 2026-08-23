# Concepts

This section explains the objects that make up ml-platform and how they relate.

| Concept | What it is |
|---|---|
| [Workspaces](workspaces.md) | The unit of isolation for everything you create |
| [Datasets](datasets.md) | Cataloged data sources that jobs read and write |
| [Jobs](jobs.md) | Units of work: data processing, training, batch inference |
| [Experiments](experiments.md) | Groupings of training runs with their metrics |
| [Models](models.md) | Registered model families and their versions |
| [Deployments](deployments.md) | Online inference: endpoints, replicas, rollouts |
| [Compute and quotas](compute.md) | Machine types and the limits that govern them |
| [Volumes](volumes.md) | Persistent workspace storage for staging and checkpoints |
| [Secrets](secrets.md) | Workspace credentials that jobs reference |
| [Data transfers](transfers.md) | Server-executed copies between locations |
| [Clusters and machines](clusters.md) | The Kubernetes clusters and nodes underneath |
| [Users and identities](identities.md) | Users, API keys, and service accounts |

## The mental model in one paragraph

You work inside a [workspace](workspaces.md).
Data you care about is registered as [datasets](datasets.md).
You submit [jobs](jobs.md) that read those datasets and run on [compute](compute.md) within the workspace's [quota](compute.md).
Training jobs attach to [experiments](experiments.md) and register their outputs as [models](models.md).
Models are either scored over whole datasets by batch inference jobs, or served behind endpoints by [deployments](deployments.md).
Everything else (secrets, volumes, transfers, clusters, identities) supports that flow.

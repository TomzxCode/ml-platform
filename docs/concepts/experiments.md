# Experiments

An experiment is a named grouping of training runs.
Training jobs attach to an experiment by name, and the platform collects the runs and their metrics under it.

Experiments answer "what did I try, and how did it go?" without leaving the CLI.

## Creating an experiment

Experiments are created from a small [spec file](../reference/specs/experiment.md):

```yaml
name: ctr-baseline
description: First pass on CTR estimation
```

```shell
mlx experiment create ctr-baseline.yaml
mlx experiment list
```

## Attaching runs

A training job attaches to an experiment through its spec's `experiment` field:

```yaml
name: ctr-baseline-run
type: training
dataset: raw-clicks
experiment: ctr-baseline
...
```

Every training job that names the experiment becomes a run of it.

## Inspecting runs and metrics

```shell
mlx experiment get ctr-baseline
```

The output lists the experiment's runs with their metrics, so you can compare hyperparameter settings side by side.

## Commands

The full command surface is in the [`mlx experiment` reference](../reference/cli/experiment.md).

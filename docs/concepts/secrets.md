# Secrets

A secret is a workspace-scoped credential that jobs reference by name: registry pull tokens, experiment tracker keys, database passwords.
The platform injects the value into the job at run time.

Secret values are write-only: the CLI can set and delete them, but never display them.
Not in listings, not in logs, not in verbose mode.

## Setting a secret

```shell
mlx secret set wandb-key --from-literal <value>
mlx secret set db-password --from-file ./password.txt
```

Exactly one source is accepted: a literal or a file.
Prefer `--from-file` (or a masked prompt) over `--from-literal` when practical, since literal values land in shell history.

## Referencing secrets in jobs

Job specs list the secrets they need by name:

```yaml
name: ctr-baseline-run
type: training
secrets: [wandb-key]
...
```

The values are injected at run time and never appear in the spec, the job record, or the logs.

## Listing and deleting

```shell
mlx secret list
mlx secret delete wandb-key
```

`secret list` prints names only.
Deleting a secret is immediate: jobs submitted afterwards that still reference it fail with an error naming the missing secret.

## Commands

The full command surface is in the [`mlx secret` reference](../reference/cli/secret.md).

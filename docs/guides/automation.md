# Automation

Everything the CLI can do interactively, it can do unattended.
This guide covers identities for automation and the conventions that make the CLI scriptable.

## Choose an identity

| Situation | Identity |
|---|---|
| CI pipeline acting as you | [API key](../concepts/identities.md#api-keys) |
| Long-lived automation owned by the team | [Service account](../concepts/identities.md#service-accounts) |

Create an API key with an expiry and use its value (shown once) where a script would use your token:

```shell
mlx auth api-key create ci-deploy --expires 30d
```

## Non-interactive login

`auth login` accepts the token and server as flags, so it works in pipelines:

```shell
mlx auth login --server https://ml.internal.example.com --token "$MLX_TOKEN"
```

Prefer reading the token from the environment or a secret store; avoid command lines that end up in logs.

## Script with JSON

With `--json` (or `--output json`), any command prints exactly one valid JSON document to stdout and nothing else:

```shell
mlx job list --json | jq -r '.[].id'
```

For plain piping over names, `--output name` prints one element per line:

```shell
mlx job list --output name
```

Branch on [exit codes](cli-conventions.md#exit-codes) instead of parsing error text:

```shell
if mlx job submit train-ctr.yaml; then
  echo "submitted"
fi
```

## A nightly scoring loop

A minimal scheduled job:

```shell
#!/usr/bin/env bash
set -euo pipefail

mlx auth login --server "$MLX_SERVER" --token "$MLX_TOKEN"

job_id=$(mlx --json batch submit score-nightly.yaml | jq -r '.id')
echo "submitted $job_id"

mlx batch logs "$job_id" --follow
mlx batch get "$job_id"
```

The script follows the job to completion and leaves the scored dataset in the catalog.

## Idempotent submissions

Submissions carry an idempotency key under the hood, so a retried `job submit` does not double-submit the same intent.
In practice: if a pipeline times out, rerunning the submit is safe.

## Revoking access

Delete the key or the service account when the automation retires:

```shell
mlx auth api-key delete ci-deploy
mlx auth service-account delete nightly-scorer
```

Requests using the revoked credentials fail authentication immediately.

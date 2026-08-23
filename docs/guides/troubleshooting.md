# Troubleshooting

Errors in the CLI name the likely cause and a fix (`error:` then `hint:`).
This page collects the common ones and what to do next.

Exit codes tell you the failure class at a glance; see [CLI conventions](cli-conventions.md#exit-codes).

## Authentication

**Exit code 2: authentication failure.**

- Your token is invalid, expired, or revoked.
- Fix: `mlx auth login` again, or check that the API key you are using still exists (`mlx auth api-key list`).

**`mlx auth status` reports no session.**

- You have not logged in on this profile, or you ran `auth logout`.
- Fix: `mlx auth login --server <url>`.

## Server

**Exit code 3: server unreachable or server error.**

- The CLI retried with backoff and gave up, or the server rejected the request.
- Fix: confirm the server URL in your profile (`mlx auth status`), check you are on the right `--profile`, and check network access.

## Workspace

**"No workspace selected."**

- Operation commands need an active workspace.
- Fix: `mlx workspace list`, then `mlx workspace select <name>`.

**Workspace deletion refused.**

- The workspace still contains resources; the error names them.
- Fix: delete the contents first, or pass `--force` if you really mean it.

## Specs and references

**"Missing field" on submit.**

- The YAML spec failed local validation; the error names the first missing field.
- Fix: compare with the [job spec reference](../reference/specs/job.md) (or the dataset/experiment/cluster equivalents).

**"Unknown dataset" / "Unknown model" / "Unknown secret".**

- The spec references a name that is not in the workspace.
- Fix: `mlx dataset list`, `mlx model list`, `mlx secret list` to see what exists; typos are the usual cause.

**"Duplicate version" on `model register`.**

- The explicit `--version` already exists for that name.
- Fix: omit `--version` to take the next one, or choose another version.

## Compute and quotas

**Quota error on job or volume submission.**

- The workspace's quota for the named resource is exhausted.
- Fix: `mlx quota list` to see used versus total; free capacity by stopping deployments or cancelling jobs, or ask an admin to raise the limit.

**Job stays `queued`.**

- The scheduler is waiting for quota or compute capacity.
- Fix: `mlx quota list` and `mlx compute get <type>` to see headroom; `mlx infra machine list --cluster <name>` shows cluster allocation.

## Jobs

**Job `failed`.**

- The container or the platform hit an error.
- Fix: `mlx job logs <job-id>` and read the tail; the error carries the likely cause and suggested fix.

**Job referencing a deleted secret or dataset.**

- Deleting a secret or dataset invalidates specs that reference it.
- Fix: recreate the secret/dataset with the same name, or update the spec.

## Transfers

**Copy fails with a permissions error before transferring.**

- The source is unreadable or the destination unwritable; both are checked before any byte moves.
- Fix: fix access on that side, then rerun.

**`--resume` says there is no state.**

- No interrupted copy is recorded for that source-destination pair.
- Fix: nothing; it ran as a fresh copy.

## Still stuck?

- `mlx <command> --help` shows exact usage for any command.
- Errors include the platform's suggested fix; if the fix points at the platform rather than your input, contact your platform team and include the request identifier from the error when present.

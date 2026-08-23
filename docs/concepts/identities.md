# Users and identities

Three kinds of identity can act on the platform: users, API keys, and service accounts.

## Users

A user is a person, organized into teams, working in one or more [workspaces](workspaces.md) with a role in each.

```shell
mlx user list
mlx user get alice@example.com
```

`user list` shows users with their team memberships; `user get` shows one user's identity, teams, and workspaces.

## API keys

An API key is a named credential for programmatic access: CI pipelines, cron jobs, scripts acting as you.

```shell
mlx auth api-key create ci-deploy --expires 30d
mlx auth api-key list
mlx auth api-key get ci-deploy
mlx auth api-key delete ci-deploy
```

The key value is displayed exactly once, at creation.
Listings and inspections show metadata (creation date, expiry, last used) but never the value.
Deleting a key revokes it immediately: requests using it fail authentication from that moment on.

## Service accounts

A service account is a non-human identity scoped to a workspace, for automation that should not be tied to a person.

```shell
mlx auth service-account create nightly-scorer
mlx auth service-account list
mlx auth service-account get nightly-scorer
mlx auth service-account delete nightly-scorer
```

Deleting a service account revokes its credentials.
Managing service accounts requires admin rights on the workspace.

## Which identity for what?

| Situation | Use |
|---|---|
| You, typing commands | Your login token (`mlx auth login`) |
| A CI pipeline running as you | An API key |
| Long-lived automation owned by the team, not a person | A service account |

See [Automation](../guides/automation.md) for the full scripting story.

## Commands

The full command surface is in the [`mlx auth`](../reference/cli/auth.md) and [`mlx user`](../reference/cli/user.md) references.

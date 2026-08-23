# mlx auth

Authentication state and machine identities.

## `auth login`

```shell
mlx auth login [--token <token>] [--server <url>]
```

Validates credentials against the API server and stores them in the active profile.

- The token comes from `--token` (for scripts) or an interactive masked prompt.
- The server URL comes from `--server`, the active profile, or a prompt.
- Rejected credentials exit 2 without retrying forever; an unreachable server is retried with backoff, then exits 3.

## `auth logout`

```shell
mlx auth logout
```

Clears the stored token but keeps the profile's server URL, so the next login needs only a token.

## `auth status`

```shell
mlx auth status
```

Prints the current identity, server, and profile, and checks the session is still valid.

## `auth api-key`

Named credentials for programmatic access acting as you.

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<name> [--expires <duration>]` | Create a key; the value prints exactly once |
| `list` | | Keys with creation date, expiry, last used |
| `get` | `<name>` | One key's metadata, never the value |
| `delete` | `<name>` | Revoke the key immediately |

## `auth service-account`

Non-human identities scoped to the workspace.

| Subcommand | Arguments | Purpose |
|---|---|---|
| `create` | `<name>` | Create a service account (admin) |
| `list` | | Service accounts with creation date and last used |
| `get` | `<name>` | One account's details and key bindings |
| `delete` | `<name>` | Remove the account and revoke its credentials |

## Related

- [Users and identities](../../concepts/identities.md)
- [Automation](../../guides/automation.md)

---
title: "mlx auth command"
status: draft
parent: FEAT-p1
---

# CLI Design: mlx auth command

## Design Principles

- Extends the `mlx` CLI conventions fixed by the FEAT-p1 plan: cyclopts, kebab-case command names, verb-first subcommands under a noun group.
- Global options, exit code scheme, and output modes are inherited from FEAT-p1 and are restated here only where the auth group specializes them.
- Secrets are never arguments in examples, never echoed, and never printed; every path that touches the token treats it as write-only.
- Interactive prompts are the fallback, not the default: flags and stored profile state win, and non-interactive sessions fail fast with a hint instead of hanging on a prompt.

## Command Tree

```
mlx
└── auth                Establish, clear, and report the CLI's identity
    ├── login           Validate credentials and store them
    ├── logout          Clear stored credentials
    └── status          Print current identity, server, and profile
```

## Commands

### `mlx auth login`

**Description:** Validates a token against the API server and stores the resulting session in the active profile (FR-1, FR-2, FR-3, FR-8).
The server URL is stored for reuse by later logins and by other command groups.

**Synopsis:**

```
mlx auth login [--token <token>] [--server <url>]
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | No spec file is submitted to this group |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--token` | string | prompt | API token; masked prompt in interactive sessions, required flag in non-interactive ones (FR-2) |
| | `--server` | URL | active profile | API server base URL; must be http(s) (FR-3) |
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx auth login
mlx auth login --token "$MLX_TOKEN" --server https://api.example.com
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Empty token after prompt | 1 | `error: token must not be empty` plus hint |
| Non-interactive without `--token` | 1 | `error: --token is required when not a TTY` plus hint |
| Malformed server URL | 1 | `error: '<value>' is not a valid http(s) URL` plus expected form |
| Credentials rejected | 2 | `error: the API server rejected the token` plus hint to check or renew it |
| Server unreachable after backoff | 3 | `error: cannot reach the API server at <url>` plus hint |

### `mlx auth logout`

**Description:** Clears the stored credentials of the active profile while keeping its server URL, so the next login needs only a token (FR-4).
Idempotent: logging out with nothing stored succeeds.

**Synopsis:**

```
mlx auth logout
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx auth logout
```

**Errors:** none beyond the global error classes; there is no server round trip.

### `mlx auth status`

**Description:** Prints the current identity, server URL, and active profile, and reports whether the stored session is still valid, without prompting or triggering a re-login (FR-5, FR-8).

**Synopsis:**

```
mlx auth status
```

**Positional arguments:**

| Argument | Required | Description |
|---|---|---|
| (none) | | |

**Options:**

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| -h | `--help` | | | Show help and exit |

**Examples:**

```bash
mlx auth status
mlx auth status --json | jq .authenticated
```

**Errors:**

| Condition | Exit | Message shape |
|---|---|---|
| Not logged in or invalid session | 2 | `error: no valid session for profile '<name>'` plus hint to run `mlx auth login` |
| Server unreachable during a validity check | 3 | same shape as login |

## Global Options

Inherited from FEAT-p1; the auth group consumes them as follows.

| Short | Long | Value | Default | Description |
|---|---|---|---|---|
| | `--profile` | name | `default` | Selects the profile whose config file holds credentials and server URL (FR-6) |
| | `--json` | | off | Single JSON document on stdout, nothing else; the token is never a field of it |
| | `--verbose` | | off | Informational logging to stderr; token redacted |
| | `--version` | | | Print version and exit |
| -h | `--help` | | | Per-command help |

## Environment Variables

| Variable | Used by | Default | Description |
|---|---|---|---|
| `NO_COLOR` | all | unset | Disables color on TTY output |
| `MLX_TOKEN` | `auth login` | unset | Proposed third token source; pending Open Question 3, not implemented in v1 |

## Configuration

- Credentials live in the active profile's config file: `<platform-config-dir>/mlx/profiles/<name>.toml`, with `[server] url` and `[auth] token` plus `[auth] expires_at` when the server reports one (FR-6, FR-8).
- Platform config dir per NFR-5: `~/.config/mlx` on Linux, `~/Library/Application Support/mlx` on macOS, `%APPDATA%\mlx` on Windows.
- The file is created with the mode restricted to the current user (0600 on POSIX, ACL-restricted on Windows).
- Precedence: CLI flags > profile config > defaults.
- `auth logout` removes the `[auth]` section and keeps `[server]`.

## Exit Codes

Inherited from the FEAT-p1 plan.

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Usage error |
| 2 | Authentication failure (rejected credentials, or `status` finding no valid session) |
| 3 | Server error (unreachable after backoff) |
| 4 | Cancelled by user |

## Output Behavior

- **stdout:** the result document (identity, server, profile); nothing else in JSON mode.
- **stderr:** prompts, progress, diagnostics, and errors.
- **Machine-readable:** `--json` prints one document per command; `auth login` emits `{"profile", "server", "user"}`, `auth logout` emits `{"profile", "status": "logged-out"}`, `auth status` emits `{"authenticated", "user", "server", "profile", "session_valid"}`.
- **TTY behavior:** key-value blocks with color on a TTY, plain otherwise; honors `NO_COLOR`; no token is ever printed on any stream.

## Help Text

```
Usage: mlx auth [COMMAND]

Establish, clear, and report the CLI's identity against the API server.

Commands:
  login   Validate credentials and store them
  logout  Clear stored credentials
  status  Print current identity, server, and profile
```

## Error Messages

- Format: `error: <message>` then `hint: <fix>` on the next line (NFR-3).
- Representative examples:

```
error: the API server rejected the token
hint: check the token value, or generate a new one and run mlx auth login again
```

```
error: cannot reach the API server at https://api.example.com
hint: check the server URL and network, then retry; see mlx auth login --help
```

## Interactive Behavior

- **Prompts:** token via a masked prompt (hidden input, no echo) when `--token` is absent and stdin is a TTY; server URL prompted only when neither `--server` nor the profile provides one (FR-2, FR-3).
- **Non-interactive:** without a TTY, missing required input is a usage error (exit 1) with a hint naming the flag, never a hang.
- **Destructive actions:** none; `logout` needs no confirmation because re-login is one command and the server URL is preserved.
- **Dry run:** not applicable.

## Example Sessions

### First interactive login

```console
$ mlx auth login
API token: ********
Server URL [https://api.example.com]:
Logged in to https://api.example.com as alice@example.com (profile: default)
```

### Non-interactive login in CI

```console
$ mlx auth login --token "$MLX_TOKEN" --server https://api.example.com --json | jq -r .user
alice@example.com
```

### Rejected credentials

```console
$ mlx auth login --token ghp_wrongtoken
error: the API server rejected the token
hint: check the token value, or generate a new one and run mlx auth login again
$ echo $?
2
```

### Unreachable server

```console
$ mlx auth login
API token: ********
retrying (2/3) connecting to https://api.example.com ...
error: cannot reach the API server at https://api.example.com
hint: check the server URL and network, then retry; see mlx auth login --help
$ echo $?
3
```

### Status then logout then status

```console
$ mlx auth status
Profile:   default
Server:    https://api.example.com
User:      alice@example.com
Session:   valid (expires 2026-08-30T12:00:00Z)
$ mlx auth logout
Logged out of profile default (server URL kept)
$ mlx auth status
error: no valid session for profile 'default'
hint: run mlx auth login to authenticate
$ echo $?
2
```

## Requirements Traceability

| Requirement | Command(s) / Option(s) | Notes |
|---|---|---|
| FR-1 | `mlx auth login` | Validates then stores on success only |
| FR-2 | `mlx auth login --token`, masked prompt | Non-interactive sessions require the flag |
| FR-3 | `mlx auth login --server`, profile, prompt | Flag > profile > prompt; http(s) validated |
| FR-4 | `mlx auth logout` | Removes `[auth]`, keeps `[server]`; idempotent |
| FR-5 | `mlx auth status` | Identity, server, profile, validity; never re-prompts |
| FR-6 | `--profile`, profile config file | File mode restricted to the current user |
| FR-7 | all commands | Exit 1/2/3 mapping per the Errors tables |
| FR-8 | `mlx auth login`, `mlx auth status` | `expires_at` stored; expired reported invalid |
| NFR-1 | all | Token never echoed, logged, or printed |
| NFR-2 | `auth login`, `auth status` | Backoff retry then exit 3 |
| NFR-3 | all | `error:` plus `hint:` format |
| NFR-4 | `auth status` | Under 500 ms when validity is answerable locally |
| NFR-5 | config storage | Per-platform config dir and restriction mechanism |
| NFR-6 | (toolchain) | Covered by the repository gates, no CLI surface |

## Out of Scope

- Token issuance and revocation (owned by the FEAT-p2 API server, FR-13 there).
- SSO, mTLS, and browser-based device flows (FEAT-p1 open question 2).
- Profile management commands (owned by FEAT-p1 FR-16); this group only reads and writes the active profile's auth fields.

## Open Questions

1. Should `auth status` exit 2 when it finds no valid session (script-friendly), or exit 0 and report state only, reserving 2 for command failure?
2. Should `MLX_TOKEN` be supported as an env-var token source ahead of the flag and prompt (mirrors requirements Open Question 3)?
3. What identity fields does the FEAT-p2 validation endpoint return (user email, display name, team), and which does `auth status` print?

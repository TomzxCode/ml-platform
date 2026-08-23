---
title: "mlx auth command spec"
status: draft
parent: FEAT-p1
---

# Spec: `mlx auth`

## Overview

The `auth` group establishes, clears, and reports the CLI's identity against the API server.
It implements FEAT-p1 FR-2.
Parent feature: FEAT-p1 (Unified ML CLI); requirement IDs stay owned by FEAT-p1.

## Commands and inputs

| Command | Arguments | Purpose |
|---|---|---|
| `auth login` | `[--token <token>] [--server <url>]` | Validate credentials and store them |
| `auth logout` | | Clear stored credentials |
| `auth status` | | Print current identity, server, and profile |

No spec file is submitted to this group; the submitted input is the flag and prompt contract below.

## Input contract

### `auth login`

| Input | Source | Required | Rules |
|---|---|---|---|
| Token | `--token`, else interactive masked prompt | Yes | Non-empty; never echoed, logged, or written to output (NFR-3) |
| Server URL | `--server`, else active profile, else prompt | Yes | Must be an http(s) URL; stored in the profile for reuse |

`--token` exists for non-interactive use (scripts, CI).
Interactive sessions default to the masked prompt when the flag is absent (FR-2).

### `auth logout` and `auth status`

No inputs beyond the global options.
`auth status` checks whether the stored session is still valid and reports it without triggering a re-login.

## Local validation

- Empty token after prompt: exit 1 with a usage error.
- Malformed server URL: exit 1 naming the malformed value and the expected form.
- Rejected credentials: exit 2, no indefinite retry (FR-2).
- Server unreachable: retry with backoff, then exit 3 with a clear message (NFR-2).

## Storage

Credentials are stored in the active profile's config file under the platform config directory, with file mode restricted to the current user.
Logout removes the token but keeps the profile's server URL so the next login needs only a token.

## Open questions

- Token expiry and refresh behavior; assumed the server returns an expiry the CLI honors (FEAT-p1 Phase 2 deliverable).

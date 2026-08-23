---
title: "mlx auth command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx auth command

## Overview

Users must establish an identity against the API server before the CLI executes any operation (FEAT-p1-FR-2).
This feature delivers the `mlx auth` command group (`login`, `logout`, `status`) that establishes, clears, and reports that identity, along with the local credential storage it relies on.
It is a child of FEAT-p1 (Unified ML CLI) and implements the auth deliverable of FEAT-p1 Phase 2; the requirement IDs it implements stay owned by FEAT-p1, and the local IDs below decompose them.

## Parent Traceability

Local IDs are qualified as `FEAT-p3-FR-n` / `FEAT-p3-NFR-n` when referenced outside this feature.

| Local ID | Parent ID | Scope decomposed |
|---|---|---|
| FR-1 to FR-3, FR-7, FR-8 | FEAT-p1-FR-2 | Authenticate the user against the API server using a token |
| FR-4, FR-5 | FEAT-p1-FR-2 | Clearing and reporting the authenticated identity |
| FR-6 | FEAT-p1-FR-2, FEAT-p1-FR-16 | Credential storage in the active named profile |
| NFR-1 | FEAT-p1-NFR-3 | Credentials never written to logs or output |
| NFR-2 | FEAT-p1-NFR-2 | Retry with backoff, clear unreachable-server failure |
| NFR-3 | FEAT-p1-NFR-1 | Actionable error messages |
| NFR-4 | FEAT-p1-NFR-4 | Interactive command speed |
| NFR-5 | FEAT-p1-NFR-5 | Cross-platform install and run |
| NFR-6 | FEAT-p1-NFR-6 | Toolchain gates and argument-surface coverage |

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Authenticate in one step interactively, and non-interactively in scripts and CI |
| ML infrastructure engineers | Auth failures that self-explain, and no credential leakage into logs or output |
| CLI implementors (FEAT-p1 Phase 2) | A stable session-storage interface every other command group codes against |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | `auth login` shall validate the supplied token against the API server and store the resulting session credentials in the active profile only on success |
| FR-2 | Must | `auth login` shall take the token from `--token` when provided, otherwise from an interactive masked prompt, and shall reject an empty token with a usage error |
| FR-3 | Must | `auth login` shall resolve the server URL from `--server`, else the active profile's stored server, else an interactive prompt, and shall reject a non-http(s) value with a usage error naming the value and the expected form |
| FR-4 | Must | `auth logout` shall remove stored credentials from the active profile while preserving the profile's stored server URL, and shall succeed when no credentials are stored |
| FR-5 | Must | `auth status` shall print the current identity, server URL, and active profile, and shall report whether the stored session is still valid without prompting or triggering a re-login |
| FR-6 | Must | The CLI shall store credentials in the active profile's config file under the platform config directory with the file mode restricted to the current user |
| FR-7 | Must | The auth commands shall exit 1 on usage errors, exit 2 on rejected credentials without indefinite retry, and exit 3 when the server stays unreachable after retrying with backoff |
| FR-8 | Should | `auth login` shall store the token expiry the server reports, and `auth status` shall report an expired session as invalid |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Security | The token shall never be echoed, logged, or written to non-interactive output by any auth command |
| NFR-2 | Must | Reliability | `auth login` and `auth status` shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable |
| NFR-3 | Must | Usability | Every auth failure shall print an actionable error naming the likely cause and one concrete fix |
| NFR-4 | Must | Performance | `auth status` shall start up and render in under 500 ms on a warm cache when the session check is served without a server round trip |
| NFR-5 | Should | Portability | Credential storage shall work on Linux, macOS, and Windows using each platform's native restriction mechanism (file mode, or ACLs where file modes do not apply) |
| NFR-6 | Should | Maintainability | The auth group shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors, and shall have pytest coverage of its argument surface |

## Constraints

- Child of FEAT-p1; the group lands as part of FEAT-p1 Phase 2 (client core) and follows the FEAT-p1 plan's command tree, exit code scheme, and global options (`--profile`, `--json`, `--verbose`).
- Token-based authentication, per the FEAT-p1 resolution; no SSO or mTLS in this feature.
- cyclopts is the CLI framework; commands and options are kebab-case.
- The group talks to the API server contract designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- No spec file is submitted to this group; the submitted input is the flag and prompt contract defined here.
- Profiles (FEAT-p1-FR-16) own the config file; auth only writes credentials and server URL into the active profile.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: successful login stores the session
      Given a running API server that accepts the token
      When the user runs auth login with that token
      Then the command exits 0 and the session credentials are stored in the active profile
    ```

    ```gherkin
    @FR-1
    Scenario: rejected credentials are not stored
      Given a running API server that rejects the token
      When the user runs auth login with that token
      Then the command exits 2, no credentials are stored, and the error names rejected credentials as the likely cause
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: interactive masked prompt
      Given an interactive session without --token
      When the user runs auth login
      Then the CLI prompts for the token with masked input and proceeds with the entered value
    ```

    ```gherkin
    @FR-2
    Scenario: non-interactive token flag
      Given a script or CI invocation
      When the user runs auth login --token <token>
      Then the CLI uses the flag value without prompting
    ```

    ```gherkin
    @FR-2
    Scenario: empty token after prompt
      Given an interactive prompt for the token
      When the user submits an empty value
      Then the CLI exits 1 with a usage error and does not contact the server
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: server flag wins and is stored
      Given a profile with a stored server URL
      When the user runs auth login --server <url> with a valid token
      Then the CLI authenticates against the flag's URL and stores it in the profile for reuse
    ```

    ```gherkin
    @FR-3
    Scenario: malformed server URL
      Given a server URL that is not an http(s) URL
      When the user runs auth login with it
      Then the CLI exits 1 naming the malformed value and the expected form
    ```

    ```gherkin
    @FR-3
    Scenario: prompt for missing server
      Given a profile without a stored server URL and an interactive session
      When the user runs auth login
      Then the CLI prompts for the server URL before authenticating
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: logout keeps the server URL
      Given a logged-in profile with a stored server URL
      When the user runs auth logout
      Then the stored credentials are removed, the server URL remains, and the command exits 0
    ```

    ```gherkin
    @FR-4
    Scenario: logout without stored credentials
      Given a profile with no stored credentials
      When the user runs auth logout
      Then the command exits 0 without error
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: status reports the session
      Given a logged-in profile with a valid session
      When the user runs auth status
      Then the CLI prints the identity, server URL, and active profile, and reports the session as valid
    ```

    ```gherkin
    @FR-5
    Scenario: status never re-prompts
      Given a profile whose stored session is expired or invalid
      When the user runs auth status
      Then the CLI reports the session as invalid and does not prompt for a token
    ```

    ```gherkin
    @FR-5
    Scenario: status honors JSON mode
      Given a logged-in profile
      When the user runs auth status --json
      Then the CLI prints a single valid JSON document with identity, server, profile, and validity, and nothing else
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: credential file is user-restricted
      Given any successful login
      When the credential file's permissions are inspected
      Then it lives under the platform config directory and only the current user can read it
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: unreachable server after retries
      Given an API server that stays unreachable
      When the user runs auth login
      Then the CLI retries with backoff, exits 3, and prints a clear message naming the server
    ```

    ```gherkin
    @FR-7
    Scenario: rejected credentials do not retry indefinitely
      Given a running API server that rejects the token
      When the user runs auth login
      Then the CLI attempts validation once per invocation and exits 2 without retrying the rejection
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: expired session reported as invalid
      Given a stored session whose server-reported expiry is in the past
      When the user runs auth status
      Then the session is reported as invalid without contacting the server
    ```

    ```gherkin
    @FR-8
    Scenario: login stores the expiry
      Given a server that reports a token expiry on validation
      When the user runs auth login with a valid token
      Then the stored session records that expiry
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: token never leaks
      Given an auth command run with verbose logging enabled
      When its output, logs, and JSON documents are inspected
      Then the token value appears nowhere
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given an API server that briefly drops the first connection
      When auth login runs
      Then the CLI retries with backoff and succeeds once the server responds
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: actionable auth errors
      Given any auth failure
      When the error is printed
      Then the message names the likely cause and one concrete fix
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: status speed
      Given a warm CLI cache and a session validity answerable without a server round trip
      When the user runs auth status
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-5**

    ```gherkin
    @NFR-5
    Scenario: credential storage on three platforms
      Given Linux, macOS, and Windows hosts with the CLI installed
      When auth login runs on each
      Then credentials are stored under each platform's config directory and restricted to the current user
    ```

- [ ] **NFR-6**

    ```gherkin
    @NFR-6
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors, and the auth group's argument surface is covered by pytest
    ```

## Conflicts

None identified yet.

## Open Questions

1. Token expiry and refresh behavior: assumed the server returns an expiry the CLI honors (FEAT-p1 Phase 2 deliverable); what happens on the first command after expiry, a re-prompt or a hard failure, is undecided.
2. Does `auth status` validate the session with a server round trip or trust the locally stored expiry (when present)? The answer bounds NFR-4 (under 500 ms) and offline behavior.
3. Should `auth login` also read the token from an environment variable (for example `MLX_TOKEN`) as a third non-interactive source?

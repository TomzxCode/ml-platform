---
title: "mlx secret command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx secret command

## Overview

MLEs need jobs that authenticate to external systems (data sources, registries, cloud APIs) without baking credentials into job specs or code.
This feature delivers the `mlx secret` command group: store a credential in the workspace (`set`), list secret names (`list`), and remove a secret (`delete`).
Values are write-only by design: no command displays, echoes, or logs a secret value, ever.
It implements FEAT-p1-FR-17 (manage workspace secrets without ever displaying values) inside the unified CLI (FEAT-p1), with FEAT-p1-NFR-3 as its governing security inheritance.
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose FEAT-p1-FR-17 for this command group and never redefine the parent's requirements.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Store job credentials with minimal friction and no shell-history footguns |
| ML infrastructure engineers | No credential leaks in output or logs (leaks are the worst kind of support load) |
| FEAT-p1 implementors | A complete secret group that satisfies FEAT-p1-FR-17 |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `secret set` command shall store a credential in the active workspace under a kebab-case name from exactly one source (`--from-literal <value>` or `--from-file <path>`) and report the stored secret's name, never its value |
| FR-2 | Must | The `secret list` command shall list the names of the secrets in the active workspace only, never any value |
| FR-3 | Must | The `secret delete` command shall remove the named secret from the workspace immediately |
| FR-4 | Must | The `secret set` command shall validate locally before any server call: exactly one source flag is supplied, the `--from-file` path exists and is readable, and the name is kebab-case; every violation exits 1 with an actionable error and no server call |
| FR-5 | Must | The `secret delete` command shall exit non-zero with an error naming the secret when no secret with that name exists, and `secret set` shall exit non-zero with a clear conflict error when the name already exists (uniqueness confirmed server-side) |
| FR-6 | Must | No `secret` command output, error message, prompt, or JSON document shall contain a secret value in any output mode or verbosity, including failures that occur after the value is in hand (server errors, transport failures) |
| FR-7 | Must | The `secret set` help text shall document the shell-history risk of `--from-literal` and recommend `--from-file` as the safer alternative |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `secret` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `secret` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Security | The CLI shall never write secret values to logs, verbose output, or non-interactive output, including exception traces and redacted echo of command arguments (inherits FEAT-p1-NFR-3) |
| NFR-4 | Must | Performance | The interactive `secret list` subcommand shall start up and render in under 500 ms on a warm cache (inherits FEAT-p1-NFR-4) |
| NFR-5 | Should | Maintainability | The `secret` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-17; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx secret set|list|delete` with no additional subcommands and no `secret get` (values are write-only by design) without a parent-feature change.
- No spec file is submitted to this group; the submitted input is the name plus exactly one value source.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client of the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- Secret storage, encryption at rest, and name uniqueness are server-side concerns; the CLI validates locally what it can and surfaces server errors otherwise.
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes) is owned by FEAT-p1 (FR-2, FR-3, FR-12) and is not restated as local requirements.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: set a secret from a literal
      Given an authenticated user in a workspace
      When the user runs secret set with a kebab-case name and a literal value
      Then the CLI reports the stored secret's name
      And the value appears nowhere in the output
    ```

    ```gherkin
    @FR-1
    Scenario: set a secret from a file
      Given an authenticated user in a workspace and a readable file containing the value
      When the user runs secret set with a kebab-case name and the file as the source
      Then the CLI reports the stored secret's name
      And the file's contents appear nowhere in the output
    ```

    ```gherkin
    @FR-1
    Scenario: set a secret and list it
      Given a secret was just set in the workspace
      When the user runs secret list
      Then the new secret's name appears in the listing
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: list shows names only
      Given the workspace contains two secrets
      When the user runs secret list
      Then both secret names appear
      And neither secret's value appears
    ```

    ```gherkin
    @FR-2
    Scenario: list an empty workspace
      Given the workspace contains no secrets
      When the user runs secret list
      Then the CLI prints an empty listing and exits 0
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: delete removes the secret immediately
      Given a secret named registry-token exists in the workspace
      When the user runs secret delete registry-token
      Then the secret no longer appears in secret list
      And the CLI exits 0 with a confirmation naming the secret
    ```

    ```gherkin
    @FR-3
    Scenario: jobs fail clearly after deletion
      Given a secret named registry-token was deleted from the workspace
      When a job referencing secret registry-token runs
      Then the job fails with a clear error naming the missing secret
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: both source flags supplied
      Given the user runs secret set with both --from-literal and --from-file
      When the command executes
      Then the CLI exits 1 with an error naming the two flags as mutually exclusive
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: neither source flag supplied
      Given the user runs secret set with a name but no source flag
      When the command executes
      Then the CLI exits 1 with an error stating exactly one source is required
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: from-file path does not exist
      Given the --from-file path does not exist
      When the user runs secret set
      Then the CLI exits 1 with an error naming the missing file
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: from-file path is not readable
      Given the --from-file path exists but is not readable
      When the user runs secret set
      Then the CLI exits 1 with a permissions error naming the file
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: name is not kebab-case
      Given the user runs secret set with the name Registry_Token
      When the command executes
      Then the CLI exits 1 with an error explaining the kebab-case rule
      And no server call is made
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: delete a secret that does not exist
      Given no secret named missing-token exists in the workspace
      When the user runs secret delete missing-token
      Then the CLI exits non-zero with an error naming missing-token
    ```

    ```gherkin
    @FR-5
    Scenario: set a duplicate name
      Given a secret named registry-token exists in the workspace
      When the user runs secret set registry-token with a new value
      Then the CLI exits non-zero with a conflict error naming registry-token
      And the stored secret is unchanged
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: JSON mode on set carries no value
      Given an authenticated user in a workspace
      When the user runs secret set with --json
      Then the CLI prints exactly one valid JSON document
      And the document does not contain the secret value
    ```

    ```gherkin
    @FR-6
    Scenario: JSON mode on list carries names only
      Given the workspace contains one secret
      When the user runs secret list with --json
      Then the CLI prints exactly one valid JSON document containing the secret names
      And the document does not contain any secret value
    ```

    ```gherkin
    @FR-6
    Scenario: errors never embed the value
      Given a secret set invocation whose value source triggers a server or file error
      When the error is printed
      Then the message names the secret and the cause without containing the value
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: help documents the literal risk
      Given the CLI is installed
      When the user runs secret set --help
      Then the help documents that --from-literal values can persist in shell history
      And it recommends --from-file as the safer alternative
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given a secret set invocation with a local validation problem
      When the command fails
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs secret --help and secret set --help
      Then each prints usage with a description of every option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs secret list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: values never leak in verbose mode
      Given a workspace with one secret set from a literal
      When the user runs secret list and secret set with verbose logging enabled
      Then the secret value appears nowhere in the output or log files
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs secret list
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-5**

    ```gherkin
    @NFR-5
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the secret group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Does `secret set` on an existing name overwrite the stored value (upsert), or exit with a conflict error as FR-5 currently requires?
2. Should an interactive masked prompt be added as a third value source so values never enter shell history at all (carried from `spec.md`)?
3. Does the CLI enforce a local maximum secret size before upload, or defer entirely to the server's limit?

---
title: "mlx data command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx data command

## Overview

MLEs need to move datasets between storage locations without routing the data through their workstation or babysitting the transfer.
This feature delivers the `mlx data copy` command: a server-executed transfer from a source location to a destination location, with progress reporting in the CLI and resumption after interruption.
It implements FEAT-p1-FR-5 (copy a dataset on request) and FEAT-p1-FR-13 (show transfer progress and resume an interrupted copy) inside the unified CLI (FEAT-p1).
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose FEAT-p1-FR-5 and FEAT-p1-FR-13 for this command group and never redefine the parent's requirements.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Move large datasets between locations with visible progress, and resume rather than restart after interruption |
| ML infrastructure engineers | Transfers that fail safely (pre-flight permission checks, no silent partial copies) and errors that avoid support load |
| FEAT-p1 implementors | A complete data group that satisfies FEAT-p1-FR-5 and FEAT-p1-FR-13 |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `data copy` command shall copy a dataset from a source location to a destination location on request, executed by the API server, and report completion with a summary (bytes copied, duration, average rate) |
| FR-2 | Must | The `data copy` command shall validate its inputs locally before any server call: both arguments parse as URIs, both use recognized schemes, and the scheme pair is supported; a validation failure exits non-zero with an error naming the failed check |
| FR-3 | Must | A copy whose source is unreadable or whose destination is unwritable shall fail with a permissions error before any data is transferred |
| FR-4 | Should | During a copy, the CLI shall show progress (bytes copied and current rate) on stderr in human mode, and shall never write progress to stdout |
| FR-5 | Should | On success, human mode shall print a one-screen summary and `--json` mode shall print exactly one JSON document to stdout (source, destination, bytes copied, duration, average rate) |
| FR-6 | Should | An interrupted copy shall leave state sufficient to resume, keyed by the source and destination pair |
| FR-7 | Should | The `--resume` flag shall continue an interrupted copy for the same source-destination pair from where it stopped, without re-copying completed parts |
| FR-8 | Should | `--resume` with no recorded state for the source-destination pair shall behave like a fresh copy and print one informational line saying so |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `data` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `data copy` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable, and a retried submission shall never start a duplicate transfer (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Performance | `data copy` local validation shall complete in under 500 ms on a warm cache before the first server call (inherits FEAT-p1-NFR-4) |
| NFR-4 | Should | Maintainability | The `data` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-5 and FEAT-p1-FR-13; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx data copy <source> <destination> [--resume]` with no additional subcommands without a parent-feature change.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client; the transfer executes server-side (FEAT-p2-FR-5), so transfer execution, resume checkpoints, and the progress reporting protocol are server-side concerns consumed through the FEAT-p2 contract.
- Until the API server exists, tests run against a contract-conformant stub.
- Recognized URI schemes in v1: `s3://`, `gs://`, `file://` (the same constant set as the dataset group).
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes) is owned by FEAT-p1 (FR-2, FR-3, FR-12) and is not restated as local requirements.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: copy a dataset
      Given the user has read access to a source dataset location and write access to a destination
      When the user runs data copy from source to destination
      Then the data appears at the destination and its contents match the source
      And the CLI prints a completion summary with bytes copied, duration, and average rate
    ```

    ```gherkin
    @FR-1
    Scenario: server-side transfer failure
      Given a copy in progress whose source becomes unavailable mid-transfer
      When the transfer fails
      Then the CLI exits non-zero with an actionable error naming the failed transfer
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: non-URI argument is rejected locally
      Given a source argument that does not parse as a URI
      When the user runs data copy with it
      Then the CLI exits non-zero naming the argument and the parse failure
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: unrecognized scheme is rejected locally
      Given a destination argument using a URI scheme the CLI does not recognize
      When the user runs data copy with it
      Then the CLI exits non-zero naming the unrecognized scheme
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: unsupported scheme pair is rejected locally
      Given a source and destination whose scheme pair is not supported
      When the user runs data copy with them
      Then the CLI exits non-zero naming the unsupported pair
      And no server call is made
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: copy with inaccessible source
      Given a source location the user cannot read
      When the user requests a copy from it
      Then the CLI exits non-zero with a permissions error before transferring anything
    ```

    ```gherkin
    @FR-3
    Scenario: copy with unwritable destination
      Given a destination location the user cannot write
      When the user requests a copy to it
      Then the CLI exits non-zero with a permissions error before transferring anything
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: progress goes to stderr only
      Given a copy in progress in human mode
      When progress is displayed
      Then bytes copied and current rate appear on stderr
      And no progress output appears on stdout
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: JSON summary document
      Given a completed copy
      When the user runs data copy with the machine-readable output flag
      Then exactly one valid JSON document is printed to stdout
      And the document contains source, destination, bytes copied, duration, and average rate
      And nothing else is printed to stdout
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: resume state is keyed by the source-destination pair
      Given two interrupted copies that share a source but have different destinations
      When the user resumes one of them
      Then only the transfer for that source-destination pair is resumed
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: interrupted copy resumes
      Given a dataset copy interrupted midway
      When the user re-runs the same copy command with --resume
      Then the CLI resumes from where it stopped without re-copying completed parts
      And the destination's final contents match the source
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: resume with no prior state
      Given no interrupted copy recorded for the source-destination pair
      When the user runs data copy with --resume
      Then a fresh copy starts
      And exactly one informational line explains that no state was found
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given a data copy invocation with an unrecognized source scheme
      When the command fails on it
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs data --help and data copy --help
      Then each prints usage with a description of every argument and option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs data copy
      Then the CLI retries with backoff and proceeds once the server responds
      And the user sees at most one informational line about the retry
    ```

    ```gherkin
    @NFR-2
    Scenario: no duplicate transfer on a retried submission
      Given a submission response was lost to a network drop
      When the client retries the submission
      Then the server continues the original transfer instead of starting a second one
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: local validation speed
      Given a warm CLI cache
      When the user runs data copy with any arguments
      Then local validation completes in under 500 ms before the first server call
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the data group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Which resume protocol does the platform adopt: a chunk ledger tracked against the transfer, or server-supported range resume over the recorded transfer (the FEAT-p1 risk register flags backend support as a risk)?
2. What is the maximum number of concurrent transfer streams within a single copy?
3. How does the CLI receive progress updates from the server (polling the transfer resource versus a streamed channel), and is that mechanism fixed by the FEAT-p2 contract?
4. What happens to the server-side transfer when the user interrupts the CLI (cancelled, paused, or continued until completion), and which party decides?

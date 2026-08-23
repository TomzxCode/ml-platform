---
title: "mlx compute command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx compute command

## Overview

Users authoring job specs need to know which machine types their workspace can use and how much of each remains under quota before they submit training or batch inference work (FEAT-p1-FR-18).
This feature delivers the `mlx compute` command group (`list`, `get`), a read-only surface that discovers the compute types available to the active workspace and inspects one type's resources and remaining quota.
It is a child of FEAT-p1 (Unified ML CLI) and implements part of FEAT-p1 Phase 3 (operation commands); the requirement IDs it implements stay owned by FEAT-p1, and the local IDs below decompose them.

## Parent Traceability

Local IDs are qualified as `FEAT-p8-FR-n` / `FEAT-p8-NFR-n` when referenced outside this feature.

| Local ID | Parent ID | Scope decomposed |
|---|---|---|
| FR-1 | FEAT-p1-FR-18, FEAT-p1-FR-3 | List the compute types available to the active workspace |
| FR-2 | FEAT-p1-FR-18 | Inspect one compute type's resources and quota |
| FR-3 | FEAT-p1-FR-18, FEAT-p1-NFR-1 | Authoritative server-side type names with closest-match errors |
| FR-4 | FEAT-p1-FR-12 | Machine-readable output for both commands |
| FR-5 | FEAT-p1 (exit code scheme) | Exit code mapping for the group |
| NFR-1 | FEAT-p1-NFR-1 | Actionable error messages |
| NFR-2 | FEAT-p1-NFR-2 | Retry with backoff, clear unreachable-server failure |
| NFR-3 | FEAT-p1-NFR-4 | Interactive command speed |
| NFR-4 | FEAT-p1-NFR-6 | Toolchain gates and argument-surface coverage |

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Discover usable machine types and remaining quota quickly when authoring job specs |
| ML infrastructure engineers | Quota visibility that preempts "why will my job not schedule" support requests |
| CLI implementors (FEAT-p1 Phase 3) | A stable, read-only surface with no mutation paths to review |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | `compute list` shall list the machine types available to the active workspace, each with its type name, GPU model and count, memory, and remaining quota |
| FR-2 | Must | `compute get <type>` shall print one machine type's details (type name, GPU model and count, memory) plus the workspace's remaining quota for that type |
| FR-3 | Must | The group shall validate the type name locally as a kebab-case string, and shall resolve unknown (but well-formed) type names server-side, failing with an error that suggests the closest matching type |
| FR-4 | Must | Both commands shall emit machine-readable output when `--json` is passed: an array of type objects for `list`, a single type object for `get`, exactly one valid JSON document and nothing else |
| FR-5 | Must | The compute commands shall exit 1 on usage errors (missing or malformed type name, unknown type name), exit 2 on authentication failure, and exit 3 when the server stays unreachable after retrying with backoff |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every compute failure shall print an actionable error naming the likely cause and one concrete fix, and unknown-type errors shall name the closest matching type |
| NFR-2 | Must | Reliability | Both commands shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable |
| NFR-3 | Must | Performance | `compute list` and `compute get` shall start up and render in under 500 ms on a warm cache, excluding server response time |
| NFR-4 | Should | Maintainability | The compute group shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors, and shall have pytest coverage of its argument surface |

## Constraints

- Child of FEAT-p1; the group lands as part of FEAT-p1 Phase 3 (operation commands) and follows the FEAT-p1 plan's command tree, exit code scheme, and global options (`--profile`, `--json`, `--verbose`).
- cyclopts is the CLI framework; commands, options, and user-visible type names are kebab-case.
- The group talks to the API server contract designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- No spec file is submitted to this group; the submitted input is the compute type name defined here.
- Read-only: the group offers no create, update, or delete path; quota limit administration belongs to the `quota` group (FEAT-p1-FR-21), while this group reports availability and remaining quota per type.
- The server is the authoritative source of type names and quota; the CLI never talks to compute infrastructure directly.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: list shows available types
      Given the active workspace has quota for at least two machine types
      When the user runs compute list
      Then every available type appears with its type name, GPU model and count, memory, and remaining quota
    ```

    ```gherkin
    @FR-1
    Scenario: list is workspace-scoped
      Given two workspaces with different compute quotas
      When the user selects workspace A and runs compute list
      Then only the types available to workspace A appear
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: get prints one type's details
      Given the active workspace has quota for machine type gpu-h100-80gb
      When the user runs compute get gpu-h100-80gb
      Then the output is a key-value block with the type name, GPU model and count, memory, and the workspace's remaining quota for that type
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: malformed type name is rejected locally
      Given a type name that is not kebab-case
      When the user runs compute get with it
      Then the CLI exits 1 with a usage error naming the expected form and does not contact the server
    ```

    ```gherkin
    @FR-3
    Scenario: unknown type suggests the closest match
      Given a well-formed type name that no available type matches
      When the user runs compute get with it
      Then the CLI exits non-zero and the error suggests the closest matching type name
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: list JSON output
      Given the active workspace has quota for two machine types
      When the user runs compute list --json
      Then the CLI prints one valid JSON document containing an array of two type objects and nothing else on stdout
    ```

    ```gherkin
    @FR-4
    Scenario: get JSON output
      Given the active workspace has quota for machine type gpu-h100-80gb
      When the user runs compute get gpu-h100-80gb --json
      Then the CLI prints one valid JSON document with a single type object and nothing else on stdout
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: usage errors exit 1
      Given a missing or malformed type name for compute get
      When the user runs the command
      Then the CLI exits 1 with a usage error
    ```

    ```gherkin
    @FR-5
    Scenario: unreachable server exits 3
      Given an API server that stays unreachable
      When the user runs compute list
      Then the CLI retries with backoff, exits 3, and prints a clear message naming the server
    ```

    ```gherkin
    @FR-5
    Scenario: authentication failure exits 2
      Given a stored session the server rejects
      When the user runs compute list
      Then the CLI exits 2 with an authentication error
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable compute errors
      Given any compute failure
      When the error is printed
      Then the message names the likely cause and one concrete fix
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given an API server that briefly drops the first connection
      When the user runs compute list
      Then the CLI retries with backoff and succeeds once the server responds
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: interactive command speed
      Given a warm CLI cache and a responsive server
      When the user runs compute list or compute get
      Then local startup and rendering complete in under 500 ms, excluding server response time
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors, and the compute group's argument surface is covered by pytest
    ```

## Conflicts

None identified yet.

## Open Questions

1. Should `compute list` support filters (for example GPU-only) in v1, or defer them to a later release?
2. If the FEAT-p2 list endpoint paginates with cursors, does `compute list` follow cursors transparently, given the expected small cardinality of machine types?
3. What exact quota fields does the contract return per type (remaining only, or used, limit, and remaining), and how are types without quota presented?

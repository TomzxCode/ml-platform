---
title: "mlx workspace command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx workspace command

## Overview

Every operation the CLI submits runs inside a workspace, the platform's unit of isolation (similar to a Kubernetes namespace).
Users need to see which workspaces they can access, choose one, and know which one is active before running operation commands.
This feature delivers the `mlx workspace` command group (`list`, `select`, `get`) that implements FEAT-p1 FR-3, with JSON output per FEAT-p1 FR-12.
Workspace membership management (FEAT-p1 FR-24) is out of scope for this feature.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Switch workspace context quickly and always know which workspace their operations target |
| ML infrastructure engineers | Uniform workspace scoping across all operation commands, so cross-workspace mistakes never generate support load |
| CLI implementors (FEAT-p1) | A single selection mechanism every other command group consumes |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The CLI shall list the workspaces the authenticated user can access, showing each workspace's name and description |
| FR-2 | Must | The CLI shall let the user select a workspace by name and persist the selection in the active profile, so later commands inherit it |
| FR-3 | Must | The CLI shall report the active workspace, or clearly state that none is selected |
| FR-4 | Must | The CLI shall scope every subsequent operation command to the selected workspace, and fail with an actionable error naming `mlx workspace select` when none is selected |
| FR-5 | Should | The CLI shall reject an unknown workspace name on `select`, exiting after the server rejects it and suggesting the closest matching workspace name |
| FR-6 | Should | The CLI shall emit machine-readable output (JSON) for every workspace command when requested |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every workspace command shall provide help text, and every failure shall print an actionable error message naming the likely cause and one concrete fix |
| NFR-2 | Should | Reliability | `list` and `select` shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable |
| NFR-3 | Should | Performance | `workspace list` and `workspace get` shall start up and render in under 500 ms on a warm cache |
| NFR-4 | Should | Maintainability | The command group shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors, and have pytest coverage of its argument surface |

## Constraints

- Command group and subcommands are fixed by the FEAT-p1 plan: `mlx workspace list|select|get`.
- Requirement IDs stay owned by FEAT-p1; this feature implements FEAT-p1 FR-3 (workspace scoping) and FEAT-p1 FR-12 (JSON output).
- The workspace roster (names, descriptions, access) is served by the API server (FEAT-p2 FR-3); the CLI checks only locally verifiable input rules and defers existence checks to the server.
- The selection is stored in the active profile's configuration (FEAT-p1 FR-16), so it is per-profile and per-server.
- Exit codes follow the CLI-wide scheme (FEAT-p1 plan): 0 success, 1 usage error, 2 authentication failure, 3 server error.
- Python with cyclopts as the CLI framework; command names are kebab-case.
- Workspace names are kebab-case strings.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: list accessible workspaces
      Given the authenticated user belongs to two workspaces
      When the user runs workspace list
      Then a table shows both workspaces with their name and description
    ```

    ```gherkin
    @FR-1
    Scenario: no accessible workspaces
      Given the authenticated user belongs to no workspace
      When the user runs workspace list
      Then the output clearly states that no workspaces are accessible
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: select persists across invocations
      Given the authenticated user has access to workspace research
      When the user runs workspace select research
      Then the command exits 0
      And a subsequent workspace get in a new invocation reports research as the active workspace
    ```

    ```gherkin
    @FR-2
    Scenario: selection is per profile
      Given two named profiles pointing at the same server
      When the user selects workspace research under profile A
      Then workspace get under profile B still reports no selection
    ```

    ```gherkin
    @FR-2
    Scenario: malformed workspace name
      Given the user invokes workspace select with the name "Research Dev"
      When the CLI validates the argument locally
      Then the CLI exits 1 with a usage error naming the expected kebab-case form
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: get reports the active workspace
      Given workspace research is selected in the active profile
      When the user runs workspace get
      Then the output names research as the active workspace
    ```

    ```gherkin
    @FR-3
    Scenario: get with no selection
      Given no workspace is selected in the active profile
      When the user runs workspace get
      Then the output clearly states that no workspace is selected
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: operations are workspace-scoped
      Given the user is authenticated and belongs to a team with two workspaces
      When the user selects workspace A and submits a training job
      Then the job is created in workspace A only
    ```

    ```gherkin
    @FR-4
    Scenario: operation without a selection
      Given no workspace is selected in the active profile
      When the user runs any operation command
      Then the CLI exits 1 and the error names running mlx workspace select as the fix
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: unknown workspace name
      Given the user has access to workspace research
      When the user runs workspace select researhc
      Then the CLI exits 3 after the server rejects the name
      And the error suggests research as the closest match
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: JSON output for list
      Given the authenticated user belongs to two workspaces
      When the user runs workspace list with the machine-readable output flag
      Then the CLI prints a single valid JSON document to stdout containing an array of two workspace objects and nothing else
    ```

    ```gherkin
    @FR-6
    Scenario: JSON output for get
      Given workspace research is selected in the active profile
      When the user runs workspace get with the machine-readable output flag
      Then the CLI prints a single valid JSON document to stdout containing one workspace object and nothing else
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable errors
      Given a workspace command that fails due to user input
      When the error is printed
      Then the message names the likely cause and one concrete fix
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs workspace list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs workspace list or workspace get
      Then the command renders its output in under 500 ms
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When uv run ruff check ., uv run ruff format --check ., uv run ty check ., and uv run pytest run
      Then all four pass with no errors
    ```

## Conflicts

None identified yet.

## Open Questions

1. Should a default workspace be auto-selected when the user belongs to exactly one?
2. What exit code does `workspace get` use when no workspace is selected, and how does JSON mode represent "none selected"?

Resolutions (2026-08-23): `workspace get` with no selection exits 0 with a clear human-readable statement, and JSON mode prints `null` (`cli-design.md`, Exit Codes and Output Behavior).

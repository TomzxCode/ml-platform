---
title: "mlx experiment command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx experiment command

## Overview

MLEs need to group their training runs into experiments so they can compare runs and their metrics instead of tracking individual job ids by hand.
This feature delivers the `mlx experiment` command group: create an experiment from a spec, list experiments in the workspace, and inspect one experiment's runs and their metrics.
It implements FEAT-p1-FR-19 (create experiments, list experiments in the workspace, inspect one experiment's runs and their metrics) inside the unified CLI (FEAT-p1).
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose FEAT-p1-FR-19 for this command group and never redefine the parent's requirements.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Group training runs and compare their metrics with minimal friction |
| ML infrastructure engineers | Errors that avoid support load, especially around experiment naming and run attachment |
| FEAT-p1 implementors | A complete experiment group that satisfies FEAT-p1-FR-19 |
| FEAT-p9 implementors | A stable attach-by-name contract for training runs |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `experiment create` command shall create an experiment in the active workspace from a YAML spec file (name, optional one-line description) and report the created experiment |
| FR-2 | Must | The `experiment list` command shall list the experiments in the active workspace, showing each experiment's name and its one-line description when present |
| FR-3 | Must | The `experiment get` command shall print one experiment's details and its runs, each run showing its identifier and its metrics, sourced from the server's run records |
| FR-4 | Must | The `experiment create` command shall validate the spec locally before any server call: the file parses as YAML, the name is present, the name is kebab-case, and the description, when present, is a single line |
| FR-5 | Must | The `experiment get` command shall exit non-zero with an error naming the experiment when no experiment with that name exists, and `experiment create` shall exit non-zero with a clear conflict error when the name already exists in the workspace (uniqueness confirmed server-side) |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `experiment` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `experiment` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Performance | Interactive `experiment` subcommands (`list`, `get`) shall start up and render in under 500 ms on a warm cache (inherits FEAT-p1-NFR-4) |
| NFR-4 | Should | Maintainability | The `experiment` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-19; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx experiment create|list|get` with no additional subcommands without a parent-feature change.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client of the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- Experiment storage, name uniqueness, run records, and their metrics are server-side concerns; the CLI validates locally what it can and surfaces server errors otherwise.
- Run metrics come from the server's run records; the CLI does not parse run logs in v1 (default for open question 1).
- Training runs attach to experiments by name through the training spec's `experiment` field (FEAT-p9); this command group creates no runs and modifies none.
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes) is owned by FEAT-p1 (FR-2, FR-3, FR-12) and is not restated as local requirements.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: create an experiment from a valid spec
      Given a valid experiment spec file and an authenticated user in a workspace
      When the user runs experiment create with the spec file
      Then the experiment appears in the workspace's experiment list
      And the CLI reports the created experiment's name
    ```

    ```gherkin
    @FR-1
    Scenario: create with a duplicate name
      Given an experiment named ctr-baseline exists in the workspace
      When the user runs experiment create with another spec using the name ctr-baseline
      Then the CLI exits non-zero with an error naming the conflict
      And the existing experiment is unchanged
    ```

    ```gherkin
    @FR-1
    Scenario: create with a missing spec file
      Given the spec file path does not exist
      When the user runs experiment create with that path
      Then the CLI exits non-zero with an error naming the missing file
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: list experiments
      Given the workspace contains two experiments, one with a description and one without
      When the user runs experiment list
      Then both experiments appear with their name
      And the description shows as one line for the experiment that has one
    ```

    ```gherkin
    @FR-2
    Scenario: list an empty workspace
      Given the workspace contains no experiments
      When the user runs experiment list
      Then the CLI renders an empty listing without error
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: inspect an experiment with runs
      Given an experiment named ctr-baseline has two runs, each with recorded metrics
      When the user runs experiment get ctr-baseline
      Then the CLI prints the experiment's name and description
      And each run appears with its identifier and its metrics
    ```

    ```gherkin
    @FR-3
    Scenario: inspect an experiment with no runs yet
      Given an experiment named ctr-baseline exists with no runs
      When the user runs experiment get ctr-baseline
      Then the CLI prints the experiment's details and an empty run listing, without error
    ```

    ```gherkin
    @FR-3
    Scenario: JSON output for machine consumption
      Given an experiment named ctr-baseline has one run with metrics
      When the user runs experiment get ctr-baseline with the JSON output flag
      Then the CLI prints a single valid JSON document containing the experiment, its runs, and each run's metrics
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: malformed YAML is rejected locally
      Given a spec file that fails YAML parsing
      When the user runs experiment create with it
      Then the CLI exits non-zero with an error naming the parse error and the file
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: missing name
      Given a spec file with no name field
      When the user runs experiment create with it
      Then the CLI exits non-zero with an error naming name as the missing field
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: name is not kebab-case
      Given a spec file whose name is CTR_Baseline
      When the user runs experiment create with it
      Then the CLI exits non-zero with an error explaining the kebab-case rule
      And no server call is made
    ```

    ```gherkin
    @FR-4
    Scenario: description spanning multiple lines
      Given a spec file whose description contains a newline
      When the user runs experiment create with it
      Then the CLI exits non-zero with an error explaining the one-line rule
      And no server call is made
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: get an experiment that does not exist
      Given no experiment named missing-exp exists in the workspace
      When the user runs experiment get missing-exp
      Then the CLI exits non-zero with an error naming missing-exp
    ```

    ```gherkin
    @FR-5
    Scenario: create reports a server-side uniqueness conflict
      Given the server rejects a create because the name is taken in the workspace
      When the user runs experiment create with that name
      Then the CLI exits non-zero with a conflict error naming the experiment
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given an experiment spec with an invalid field
      When experiment create fails on it
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs experiment --help and experiment create --help
      Then each prints usage with a description of every option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs experiment list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs experiment list or experiment get
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the experiment group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Do run metrics come from the server's run records only, or can the CLI also parse run logs (default recorded in Constraints: server run records only in v1)?
2. Does submitting a training spec that references a never-created experiment name auto-create the experiment or fail with an error (decision shared with FEAT-p9; default: fail naming the experiment)?
3. Must `experiment get` paginate runs for experiments with many runs, following the cursor convention, or does the server return all runs of one experiment in a single response in v1?

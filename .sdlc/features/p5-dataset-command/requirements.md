---
title: "mlx dataset command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx dataset command

## Overview

MLEs need to catalog the datasets their jobs reference so that specs can name a dataset instead of a raw storage path.
This feature delivers the `mlx dataset` command group: create a catalog entry from a spec, list datasets, inspect one dataset, and delete a catalog entry.
It implements FEAT-p1-FR-20 (manage dataset catalog entries) inside the unified CLI (FEAT-p1).
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose FEAT-p1-FR-20 for this command group and never redefine the parent's requirements.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Register, find, and inspect datasets with minimal friction |
| ML infrastructure engineers | Deletion that never destroys data, and errors that avoid support load |
| FEAT-p1 implementors | A complete dataset group that satisfies FEAT-p1-FR-20 |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `dataset create` command shall register a dataset in the active workspace's catalog from a YAML spec file (name, location, format, optional description and labels) and report the created entry |
| FR-2 | Must | The `dataset list` command shall list the datasets in the active workspace, showing each entry's name, format, and one-line description when present |
| FR-3 | Must | The `dataset get` command shall print one dataset's details: name, location, format, version, description, and labels |
| FR-4 | Must | The `dataset delete` command shall remove the dataset's catalog entry only, and shall never modify or delete the data at the dataset's location |
| FR-5 | Must | The `dataset create` command shall validate the spec locally before any server call: the file parses as YAML, every required field is present, the format is a known catalog format, the location uses a recognized URI scheme, and the name is kebab-case |
| FR-6 | Must | The `dataset get` and `dataset delete` commands shall exit non-zero with an error naming the dataset when no entry with that name exists, and `dataset create` shall exit non-zero with a clear conflict error when the name already exists (uniqueness confirmed server-side) |
| FR-7 | Should | The `dataset list` command shall support filtering the listing by label |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `dataset` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `dataset` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Performance | Interactive `dataset` subcommands (`list`, `get`) shall start up and render in under 500 ms on a warm cache (inherits FEAT-p1-NFR-4) |
| NFR-4 | Should | Maintainability | The `dataset` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-20; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx dataset create|list|get|delete` with no additional subcommands without a parent-feature change.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client of the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- Catalog storage and name uniqueness are server-side concerns; the CLI validates locally what it can and surfaces server errors otherwise.
- Known catalog formats in v1: parquet, csv, jsonl.
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes) is owned by FEAT-p1 (FR-2, FR-3, FR-12) and is not restated as local requirements.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: create a dataset from a valid spec
      Given a valid dataset spec file and an authenticated user in a workspace
      When the user runs dataset create with the spec file
      Then the dataset appears in the workspace catalog
      And the CLI reports the created entry's name and version
    ```

    ```gherkin
    @FR-1
    Scenario: create with a duplicate name
      Given a dataset named raw-clicks exists in the workspace
      When the user runs dataset create with another spec using the name raw-clicks
      Then the CLI exits non-zero with an error naming the conflict
      And the existing catalog entry is unchanged
    ```

    ```gherkin
    @FR-1
    Scenario: create with a missing spec file
      Given the spec file path does not exist
      When the user runs dataset create with that path
      Then the CLI exits non-zero with an error naming the missing file
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: list datasets
      Given the workspace contains two datasets, one with a description and one without
      When the user runs dataset list
      Then both datasets appear with their name and format
      And the description shows as one line for the dataset that has one
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: get one dataset
      Given a dataset named raw-clicks exists with location, format, version, description, and labels
      When the user runs dataset get raw-clicks
      Then the CLI prints the dataset's name, location, format, version, description, and labels
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: delete removes only the catalog entry
      Given a dataset named raw-clicks exists and its location holds data
      When the user runs dataset delete raw-clicks
      Then the entry no longer appears in dataset list or dataset get
      And the data at the location is untouched
    ```

    ```gherkin
    @FR-4
    Scenario: jobs fail with a clear error after deletion
      Given a dataset named raw-clicks was deleted from the catalog
      When a job spec referencing dataset raw-clicks is submitted
      Then submission fails with an error naming the missing dataset
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: malformed YAML is rejected locally
      Given a spec file that fails YAML parsing
      When the user runs dataset create with it
      Then the CLI exits non-zero with an error naming the parse error and the file
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: missing required field
      Given a spec file whose location field is absent
      When the user runs dataset create with it
      Then the CLI exits non-zero with an error naming location as the first missing field
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: unknown format
      Given a spec file with format xml
      When the user runs dataset create with it
      Then the CLI exits non-zero with an error listing the accepted formats parquet, csv, and jsonl
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: unrecognized location scheme
      Given a spec file whose location uses a URI scheme the CLI does not recognize
      When the user runs dataset create with it
      Then the CLI exits non-zero with an error naming the unrecognized scheme
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: name is not kebab-case
      Given a spec file whose name is Raw_Clicks
      When the user runs dataset create with it
      Then the CLI exits non-zero with an error explaining the kebab-case rule
      And no server call is made
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: get a dataset that does not exist
      Given no dataset named missing-set exists in the workspace
      When the user runs dataset get missing-set
      Then the CLI exits non-zero with an error naming missing-set
    ```

    ```gherkin
    @FR-6
    Scenario: delete a dataset that does not exist
      Given no dataset named missing-set exists in the workspace
      When the user runs dataset delete missing-set
      Then the CLI exits non-zero with an error naming missing-set
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: filter the listing by label
      Given the workspace contains one dataset labeled daily and one labeled nightly
      When the user runs dataset list filtered to the label daily
      Then only the dataset labeled daily appears
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given a dataset spec with an invalid field
      When dataset create fails on it
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs dataset --help and dataset create --help
      Then each prints usage with a description of every option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs dataset list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs dataset list or dataset get
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the dataset group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Does `dataset delete` require interactive confirmation, a `--yes` flag, or neither?
2. Does the catalog version dataset entries, so re-creating the same name bumps the version instead of erroring (interacts with the FR-1 duplicate-name conflict error)?
3. Does `dataset create` verify the location is reachable at creation time, and if so, client-side or server-side?

---
title: "mlx model command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx model command

## Overview

MLEs need to register externally trained model artifacts in the registry so batch inference and online inference can reference them, and to discover what is registered before writing specs.
This feature delivers the `mlx model` command group: register an externally trained artifact under a name, list registered models, and inspect one model's versions and artifact locations.
It implements FEAT-p1-FR-7 (list, inspect, and register models) inside the unified CLI (FEAT-p1).
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose FEAT-p1-FR-7 for this command group and never redefine the parent's requirements.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Register external artifacts and find a model's name:version with minimal friction |
| ML infrastructure engineers | Registry reads that never mutate artifacts, and errors that avoid support load |
| FEAT-p1 implementors | A complete model group that satisfies FEAT-p1-FR-7 |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `model register` command shall register an externally trained model artifact in the active workspace's registry from an artifact path and a `--name`, with an optional explicit `--version`, and report the registered model name and assigned version |
| FR-2 | Must | The `model register` command shall default the version to the next version under the name when `--version` is omitted, and shall exit non-zero with a duplicate-version error when an explicit `--version` already exists under that name (uniqueness confirmed server-side) |
| FR-3 | Must | The `model list` command shall list the models registered in the active workspace, showing each model's name and version count |
| FR-4 | Must | The `model get` command shall print one model's versions, one block per version with its artifact location and creation time |
| FR-5 | Must | The `model register` command shall validate locally before any server call: the artifact path exists locally or uses a recognized URI scheme, the name is kebab-case, and an explicit version matches the registry's version format |
| FR-6 | Must | The `model get` command shall exit non-zero with an error naming the model when no model with that name exists |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `model` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `model` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Performance | Interactive `model` subcommands (`list`, `get`) shall start up and render in under 500 ms on a warm cache (inherits FEAT-p1-NFR-4) |
| NFR-4 | Should | Maintainability | The `model` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-7; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx model register|list|get` with no additional subcommands without a parent-feature change.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client of the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- Registry storage, version assignment, and version uniqueness are server-side concerns; the CLI validates locally what it can and surfaces server errors otherwise.
- No spec file is submitted to this group; the submitted inputs are the artifact path, the name, and the optional version.
- Versions produced by training jobs are registered server-side (FEAT-p2-FR-6); this group registers externally trained artifacts and reads every version regardless of producer.
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes, list output formats) is owned by FEAT-p1 (FR-2, FR-3, FR-12, FR-28) and is not restated as local requirements.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: register an external artifact
      Given a model artifact at an accessible path and an authenticated user in a workspace
      When the user runs model register with the path and name churn-rf
      Then the model churn-rf appears in the workspace registry
      And the CLI reports the registered name and assigned version
    ```

    ```gherkin
    @FR-1
    Scenario: register with an explicit version
      Given a model artifact at an accessible path
      When the user runs model register with the path, name churn-rf, and version 3
      Then the CLI reports churn-rf registered at version 3
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: version defaults to the next one under the name
      Given the model churn-rf exists with its latest version at 2
      When the user registers another artifact under the name churn-rf without a version
      Then the CLI reports the new version as 3
    ```

    ```gherkin
    @FR-2
    Scenario: duplicate explicit version is rejected
      Given the model churn-rf has version 2 registered
      When the user registers an artifact under churn-rf with explicit version 2
      Then the CLI exits non-zero with a duplicate-version error naming churn-rf and version 2
      And the existing version 2 is unchanged
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: list models with version counts
      Given the workspace contains one model with two versions and one model with one version
      When the user runs model list
      Then both models appear with their name and version count
    ```

    ```gherkin
    @FR-3
    Scenario: list an empty registry
      Given the workspace contains no registered models
      When the user runs model list
      Then the CLI renders an empty listing and exits zero
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: inspect one model's versions
      Given the model churn-rf exists with two versions registered
      When the user runs model get churn-rf
      Then the CLI prints one block per version with its artifact location and creation time
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: local artifact path does not exist
      Given the artifact path does not exist on the local filesystem and uses no URI scheme
      When the user runs model register with that path
      Then the CLI exits non-zero with an error naming the missing path
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: unrecognized artifact URI scheme
      Given an artifact path whose URI scheme the CLI does not recognize
      When the user runs model register with that path
      Then the CLI exits non-zero with an error naming the unrecognized scheme
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: name is not kebab-case
      Given a registration with name Churn_RF
      When the user runs model register with that name
      Then the CLI exits non-zero with an error explaining the kebab-case rule
      And no server call is made
    ```

    ```gherkin
    @FR-5
    Scenario: explicit version does not match the registry format
      Given the registry versions are integer sequence numbers
      When the user runs model register with version v1.2.3
      Then the CLI exits non-zero with an error naming the accepted version format
      And no server call is made
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: get a model that does not exist
      Given no model named missing-model exists in the workspace
      When the user runs model get missing-model
      Then the CLI exits non-zero with an error naming missing-model
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given a model registration with an invalid input
      When model register fails on it
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs model --help and model register --help
      Then each prints usage with a description of every option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs model list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs model list or model get
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the model group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Version scheme: integer sequence versus semver, and its default bump rule when `--version` is omitted?
2. Does registering from a local path upload the artifact to artifact storage, or record the path only?
3. Does `model get` display lineage (the producing training job) for server-registered versions, given the FEAT-p2 inspect response carries it?

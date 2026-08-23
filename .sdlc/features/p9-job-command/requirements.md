---
title: "mlx job command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx job command

## Overview

MLEs need to run work on the platform: process data, train models, and score datasets at scale.
This feature delivers the `mlx job` command group: submit a job defined by a YAML spec, list jobs, inspect one job, stream its logs, and cancel a job.
It implements FEAT-p1-FR-4, FR-6, FR-8, FR-11, and FR-14 inside the unified CLI (FEAT-p1).
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose those parent requirements for this command group and never redefine them.
This feature owns the normative job spec schema (vocabulary: Job spec) that FEAT-p10 (`mlx batch`) shares without forking.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Submit and track processing, training, and batch inference work with minimal friction |
| ML infrastructure engineers | Specs validated before submission, and errors that avoid support load |
| FEAT-p1 implementors | A complete job group that satisfies FEAT-p1-FR-4, FR-6, FR-8, FR-11, and FR-14 |
| FEAT-p10 implementors | A shared, normative job spec schema so the batch fast path never drifts |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `job submit` command shall submit a job from a YAML spec file discriminated by `type` (`processing`, `training`, or `batch-inference`), and shall report the server-assigned job id (FEAT-p1-FR-4, FR-6, FR-8) |
| FR-2 | Must | The `job submit` command shall validate the spec locally before any server call: the file parses as YAML, every required common and per-type field is present per `type`, `type` and `compute.type` hold valid values, and references are structurally valid; every failure exits non-zero naming the offending field (FEAT-p1-NFR-1) |
| FR-3 | Must | The `job submit` command shall surface the server's reference confirmation failures, with errors naming each unknown dataset, model, secret, or compute type reference (FEAT-p1-FR-4, FR-6) |
| FR-4 | Must | The `job list` command shall list the jobs in the active workspace showing each job's id, name, type, and state, and shall filter the listing by `--type` and `--state` (FEAT-p1-FR-11) |
| FR-5 | Must | The `job get` command shall print one job's current state and details by id, and shall exit non-zero with an error naming the job when no job with that id exists (FEAT-p1-FR-11) |
| FR-6 | Must | The `job logs` command shall print a job's accumulated logs and exit, exiting non-zero with an error naming the job when no job with that id exists; with `--follow` it shall keep the stream open and print lines as they are produced until the job reaches a terminal state or the user interrupts (FEAT-p1-FR-11) |
| FR-7 | Must | The `job cancel` command shall request cancellation of one job by id and report the outcome, and shall exit non-zero with an error naming the job when it does not exist or already holds a terminal state (FEAT-p1-FR-14) |
| FR-8 | Must | The job spec schema (common fields plus the per-type fields for `processing`, `training`, and `batch-inference`) shall be defined once in this feature and shared unchanged with FEAT-p10's `batch submit` |
| FR-9 | Must | The `job submit` command shall accept secret references by name only, and secret values shall never appear in the spec, the submission request, or any `job` command output (FEAT-p1-FR-17) |
| FR-10 | Should | The `job submit` command shall be safe to retry: a submission retried after a transient failure (NFR-2) shall not create a second job |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `job` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `job` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Security | The `job` subcommands shall never print secret values or credentials, including in verbose and JSON output (inherits FEAT-p1-NFR-3) |
| NFR-4 | Must | Performance | Interactive `job` subcommands (`list`, `get`) shall start up and render in under 500 ms on a warm cache (inherits FEAT-p1-NFR-4); the streaming command (`logs --follow`) is excluded from this budget |
| NFR-5 | Should | Maintainability | The `job` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-4, FR-6, FR-8, FR-11, and FR-14; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx job submit|list|get|logs|cancel` with no additional subcommands without a parent-feature change.
- This feature owns the normative YAML job spec schema; FEAT-p10 (`mlx batch`) references it and never forks it.
- The job state model (queued, running, succeeded, failed, cancelled) and every state transition are server-side concerns owned by FEAT-p2; the CLI reports state and never derives it.
- Placement, dispatch, quota enforcement, and log retention are server-side; the CLI is a client and never talks to compute infrastructure.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client of the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes) is owned by FEAT-p1 (FR-2, FR-3, FR-12) and is not restated as local requirements.
- Compute type names come from `mlx compute list` (FEAT-p1-FR-18); the CLI validates structure locally and the server confirms existence.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: submit a training job
      Given a valid training spec file and an authenticated user in a workspace
      When the user runs job submit with the spec file
      Then the CLI reports the new job's id
      And the job appears in job list with type training
    ```

    ```gherkin
    @FR-1
    Scenario: submit a processing job
      Given a valid processing spec referencing an existing dataset
      When the user runs job submit with the spec file
      Then the CLI reports the new job's id
      And the job appears in job list with type processing
    ```

    ```gherkin
    @FR-1
    Scenario: submit a batch inference job
      Given a valid batch-inference spec referencing a registered model and dataset
      When the user runs job submit with the spec file
      Then the CLI reports the new job's id
      And the job appears in job list with type batch-inference
    ```

    ```gherkin
    @FR-1
    Scenario: submit with a missing spec file
      Given the spec file path does not exist
      When the user runs job submit with that path
      Then the CLI exits non-zero with an error naming the missing file
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: malformed YAML is rejected locally
      Given a spec file that fails YAML parsing
      When the user runs job submit with it
      Then the CLI exits non-zero with an error naming the parse error and the file
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: missing per-type required field
      Given a training spec whose dataset field is absent
      When the user runs job submit with it
      Then the CLI exits non-zero with an error naming dataset as the first missing field
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: unknown job type
      Given a spec whose type is fine-tuning
      When the user runs job submit with it
      Then the CLI exits non-zero with an error listing the accepted types processing, training, and batch-inference
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: invalid compute type syntax
      Given a spec whose compute.type is an empty string
      When the user runs job submit with it
      Then the CLI exits non-zero with an error naming compute.type and pointing to mlx compute list
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: malformed model reference
      Given a batch-inference spec whose model is ctr-estimator without a version
      When the user runs job submit with it
      Then the CLI exits non-zero with an error naming the model field and the name:version rule
      And no server call is made
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: unknown dataset reference is named
      Given a training spec referencing a dataset that does not exist
      When the user runs job submit with it
      Then the CLI exits non-zero and names the unknown dataset reference
    ```

    ```gherkin
    @FR-3
    Scenario: unknown model reference is named
      Given a batch-inference spec referencing model ctr-estimator:9 when the registry holds no version 9
      When the user runs job submit with it
      Then the CLI exits non-zero and names the model reference
    ```

    ```gherkin
    @FR-3
    Scenario: unknown secret reference is named
      Given a training spec whose secrets list names a secret absent from the workspace
      When the user runs job submit with it
      Then the CLI exits non-zero and names the missing secret
    ```

    ```gherkin
    @FR-3
    Scenario: unknown compute type is named
      Given a spec whose compute.type is not among the workspace's compute types
      When the user runs job submit with it
      Then the CLI exits non-zero and names the unknown compute type
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: list jobs
      Given the workspace contains jobs of different types and states
      When the user runs job list
      Then every job appears with its id, name, type, and state
    ```

    ```gherkin
    @FR-4
    Scenario: filter the listing by type
      Given the workspace contains training and processing jobs
      When the user runs job list filtered to the type training
      Then only training jobs appear
    ```

    ```gherkin
    @FR-4
    Scenario: filter the listing by state
      Given the workspace contains running and succeeded jobs
      When the user runs job list filtered to the state running
      Then only running jobs appear
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: get one job
      Given a job in a running state
      When the user runs job get with its id
      Then the CLI prints the job's current state, type, and details
    ```

    ```gherkin
    @FR-5
    Scenario: get a job that does not exist
      Given no job with id j-missing exists
      When the user runs job get j-missing
      Then the CLI exits non-zero with an error naming j-missing
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: print accumulated logs
      Given a job that has produced log lines and holds a terminal state
      When the user runs job logs with its id
      Then the CLI prints the accumulated log lines and exits zero
    ```

    ```gherkin
    @FR-6
    Scenario: logs of a job with no lines yet
      Given a job that has produced no log lines
      When the user runs job logs with its id
      Then the CLI prints no log lines and exits zero
    ```

    ```gherkin
    @FR-6
    Scenario: logs of a job that does not exist
      Given no job with id j-missing exists
      When the user runs job logs j-missing
      Then the CLI exits non-zero with an error naming j-missing
    ```

    ```gherkin
    @FR-6
    Scenario: follow a running job's logs
      Given a running job producing log lines
      When the user runs job logs with its id and --follow
      Then log lines are printed as they are produced
      And the stream stays open until the job reaches a terminal state or the user interrupts
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: cancel a running job
      Given a submitted job in a running state
      When the user requests cancellation with its id
      Then the job reaches a cancelled state and no further logs are produced
    ```

    ```gherkin
    @FR-7
    Scenario: cancel a job that does not exist
      Given no job with id j-missing exists
      When the user requests cancellation of j-missing
      Then the CLI exits non-zero with an error naming j-missing
    ```

    ```gherkin
    @FR-7
    Scenario: cancel a job in a terminal state
      Given a job in the succeeded state
      When the user requests cancellation with its id
      Then the CLI exits non-zero with an error naming the job and its current state
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: batch submit shares the job spec schema
      Given the job spec schema module owned by this feature
      When FEAT-p10's batch submit validates a batch-inference spec
      Then it validates through the same schema module and no forked copy of the schema exists
    ```

- [ ] **FR-9**

    ```gherkin
    @FR-9
    Scenario: secrets are names only
      Given a training spec whose secrets list one secret name
      When the user runs job submit with it and then job get on the created job
      Then the CLI output names the secret
      And no secret value appears in any output
    ```

- [ ] **FR-10**

    ```gherkin
    @FR-10
    Scenario: retried submission creates one job
      Given the API server briefly drops the first submission request
      When the user runs job submit and the CLI retries with backoff
      Then exactly one job is created and the CLI reports that job's id
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given a job spec with an invalid field
      When job submit fails on it
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs job --help and job submit --help
      Then each prints usage with a description of every option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs job list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: secret values never leak
      Given a job spec referencing a secret and an authenticated session
      When the user runs job submit, job list, job get, and job logs with verbose logging enabled
      Then no secret value or credential appears anywhere in the output or log files
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs job list or job get
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-5**

    ```gherkin
    @NFR-5
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the job group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Does `code` also accept a source repository reference in addition to a container image?
2. What are the mount path semantics per input, and does the FEAT-p2 contract expose them?
3. Is `hyperparameters` constrained to string and number values, or is any YAML value accepted?
4. Which log streaming transport does the CLI consume in v1: REST SSE, gRPC, or both (the FEAT-p2 contract sketches both)?
5. Does `job logs --follow` on a queued job hold the stream open until lines appear, or exit with a note?
6. Does `job get` echo the submitted spec's `env` values, and if so how are sensitive values protected?

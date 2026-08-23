---
title: "mlx batch command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx batch command

## Overview

MLEs need to run a registered model over a dataset at scale (nightly scoring, backfills) without learning the general job spec machinery (FEAT-p1-FR-8) or generic job listing filters to track it (FEAT-p1-FR-11).
This feature delivers the `mlx batch` command group (`submit`, `list`, `get`, `logs`, `cancel`): the batch-inference fast path that submits a batch inference spec and mirrors the job lifecycle commands for batch jobs.
It is a child of FEAT-p1 (Unified ML CLI) and lands as a slice of FEAT-p1 Phases 3 and 4; the requirement IDs it implements stay owned by FEAT-p1, and the local IDs below decompose them.
The batch-inference spec schema itself is owned by FEAT-p9 (`mlx job`) and is shared, not forked: this feature consumes it so the two submission paths never drift.

## Parent Traceability

Local IDs are qualified as `FEAT-p10-FR-n` / `FEAT-p10-NFR-n` when referenced outside this feature.

| Local ID | Parent ID | Scope decomposed |
|---|---|---|
| FR-1 to FR-4 | FEAT-p1-FR-8 | Submit batch inference jobs that run a registered model over a dataset |
| FR-5 to FR-9 | FEAT-p1-FR-11 | Report job state for batch jobs (list, inspect, stream logs) |
| FR-10, FR-11 | FEAT-p1-FR-14 | Cancel a submitted batch job on request |
| FR-12 | FEAT-p1-FR-12 | Machine-readable (JSON) output when requested |
| NFR-1 | FEAT-p1-NFR-1 | Actionable error messages |
| NFR-2 | FEAT-p1-NFR-2 | Retry with backoff, clear unreachable-server failure |
| NFR-3 | FEAT-p1-NFR-3 | Credentials and secret material never written to logs or output |
| NFR-4 | FEAT-p1-NFR-4 | Interactive command speed |
| NFR-5 | FEAT-p1-NFR-5 | Cross-platform install and run |
| NFR-6 | FEAT-p1-NFR-6 | Toolchain gates and argument-surface coverage |
| NFR-7 | FEAT-p1-FR-8, FEAT-p1-NFR-6 | Shared spec schema and submission machinery with FEAT-p9 |

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Score datasets through one short command path, and track the job the same way they track every other job |
| ML infrastructure engineers | One submission machinery and one spec schema to maintain (no forked batch variant), and errors that self-explain |
| FEAT-p9 implementors | The batch group consumes the job spec schema and submission machinery they own, without changing either |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | `batch submit <spec-file>` shall validate the submitted spec locally against the batch-inference branch of the shared job spec schema (FEAT-p9) before any server call, and shall exit 1 naming the offending field on any schema, enum, or structural-reference failure |
| FR-2 | Must | `batch submit` shall reject a spec whose `type` is not `batch-inference` with a usage error naming the actual type and pointing the user at `job submit` for other job types |
| FR-3 | Must | `batch submit` shall confirm the model `name:version` reference and every input dataset reference with the server before submission, and shall exit non-zero naming any unknown reference |
| FR-4 | Must | `batch submit` shall submit the accepted spec to the active workspace and print the returned job id |
| FR-5 | Must | `batch list` shall list only batch inference jobs in the active workspace with id, name, and state, and its output shall equal `job list --type batch-inference` (both listings follow the server's cursor to exhaustion, per FEAT-p9's landed model) |
| FR-6 | Must | `batch list --state <state>` shall filter the listing to jobs in the given state, rejecting an unknown state value with a usage error |
| FR-7 | Must | `batch get <id>` shall print one batch job's current state, and shall exit non-zero naming the id when no such job exists |
| FR-8 | Must | `batch logs <id>` shall print the job's logs, and `--follow` shall keep the stream open, appending lines as they are produced until the job reaches a terminal state or the user interrupts |
| FR-9 | Must | `batch get`, `batch logs`, and `batch cancel` shall accept the same job ids as their `job` counterparts (FEAT-p1 plan, command reference) |
| FR-10 | Should | `batch cancel <id>` shall request cancellation of the named batch job |
| FR-11 | Should | `batch cancel` on a job already in a terminal state (succeeded, failed, or cancelled) shall exit non-zero with an error naming the job's current state |
| FR-12 | Should | Every batch command shall emit exactly one valid JSON document to stdout and nothing else when run with `--json` |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every batch command failure shall print an actionable error naming the likely cause and one concrete fix, including spec-validation failures that name the offending field |
| NFR-2 | Must | Reliability | Every batch command shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable |
| NFR-3 | Must | Security | The batch commands shall never echo, log, or print secret values or environment-variable values supplied in the spec, in human or JSON output |
| NFR-4 | Must | Performance | `batch --help` and the local-validation failure path of `batch submit` shall start up and render in under 500 ms on a warm cache, excluding server round trips |
| NFR-5 | Should | Portability | The batch group shall install and run on Linux, macOS, and Windows via the standard Python toolchain, with no platform-specific code |
| NFR-6 | Should | Maintainability | The batch group shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors, and shall have pytest coverage of its argument surface |
| NFR-7 | Must | Maintainability | The batch-inference spec schema, its local validation, and the submission machinery shall be the same code as FEAT-p9's `job submit`, restricted to the batch-inference branch, with no forked copy |

## Constraints

- Child of FEAT-p1; the command surface is fixed by the FEAT-p1 plan's command reference and the pre-seeded spec for this group (`submit`, `list`, `get`, `logs`, `cancel`), so no separate CLI-design artifact is produced.
- The batch-inference spec schema is owned normatively by FEAT-p9; this feature references it and shall not redefine or fork it.
- The group consumes the FEAT-p2 API server contract (job submission, listing, inspection, log streaming, cancellation); until that server exists, tests run against a contract-conformant stub.
- cyclopts is the CLI framework; commands and options are kebab-case; global options (`--profile`, `--json`, `--verbose`) and the exit code scheme (0, 1, 2, 3, 4) are inherited from FEAT-p1.
- Reference confirmation (model version, dataset), placement, and dispatch are server-owned; the CLI never talks to compute infrastructure directly.
- All job types share one lifecycle (queued, running, succeeded, failed, cancelled); the batch group observes it and triggers cancellation, it does not define it.
- stdout carries results only; prompts, progress, and diagnostics go to stderr (FEAT-p1 conventions).

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: spec missing a required field
      Given a batch inference spec whose output.dataset is absent
      When the user runs batch submit with it
      Then the CLI exits 1, names output.dataset as the offending field, and contacts no server
    ```

    ```gherkin
    @FR-1
    Scenario: malformed model reference syntax
      Given a batch inference spec whose model field is not in name:version form
      When the user runs batch submit with it
      Then the CLI exits 1 naming the model field and the expected name:version form
    ```

    ```gherkin
    @FR-1
    Scenario: unparsable spec file
      Given a file that does not parse as YAML
      When the user runs batch submit with it
      Then the CLI exits 1 with an error naming the file and the parse failure
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: training spec is rejected
      Given a job spec with type training
      When the user runs batch submit with it
      Then the CLI exits 1 naming the actual type and suggesting job submit
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: unknown model version
      Given a structurally valid spec referencing a model version not in the registry
      When the user runs batch submit with it
      Then the CLI exits non-zero naming the unknown model reference
      And no job is created
    ```

    ```gherkin
    @FR-3
    Scenario: unknown input dataset
      Given a structurally valid spec whose input dataset is not in the catalog
      When the user runs batch submit with it
      Then the CLI exits non-zero naming the unknown dataset reference
      And no job is created
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: successful submission
      Given a valid batch inference spec referencing a registered model and a cataloged dataset
      When the user runs batch submit with it
      Then the CLI exits 0, prints a job id, and the job appears in batch list
    ```

    ```gherkin
    @FR-4
    Scenario: resubmission is safe
      Given a just-submitted batch job
      When the user resubmits the identical spec and the server applies idempotency
      Then the CLI prints a job id and exits 0
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: list shows only batch jobs
      Given the workspace contains one training job and two batch inference jobs
      When the user runs batch list
      Then both batch jobs appear with id, name, and state, and the training job does not
    ```

    ```gherkin
    @FR-5
    Scenario: equivalence with job list
      Given a workspace with jobs of several types
      When the user runs batch list and job list --type batch-inference
      Then both listings show the same jobs with the same columns
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: filter by state
      Given the workspace contains batch jobs in queued, running, and succeeded states
      When the user runs batch list --state running
      Then only the running batch jobs appear
    ```

    ```gherkin
    @FR-6
    Scenario: unknown state value
      Given any workspace state
      When the user runs batch list --state finishing
      Then the CLI exits 1 naming the invalid value and the accepted states
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: get shows current state
      Given a batch job in a running state
      When the user runs batch get with its id
      Then the CLI prints the job's current state
    ```

    ```gherkin
    @FR-7
    Scenario: unknown job id
      Given no job with id j-999 in the workspace
      When the user runs batch get j-999
      Then the CLI exits non-zero naming the unknown id
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: print collected logs
      Given a finished batch job that produced log lines
      When the user runs batch logs with its id
      Then the CLI prints the stored logs and exits 0
    ```

    ```gherkin
    @FR-8
    Scenario: follow appends lines until terminal state
      Given a running batch job that keeps producing log lines
      When the user runs batch logs <id> --follow
      Then the CLI appends lines as they are produced and exits when the job reaches a terminal state
    ```

- [ ] **FR-9**

    ```gherkin
    @FR-9
    Scenario: job ids are interchangeable
      Given a batch job submitted via batch submit
      When the user runs job get, job logs, and job cancel with its id
      Then each command operates on that same job
    ```

    ```gherkin
    @FR-9
    Scenario: batch commands reject nothing the job commands accept
      Given a batch job submitted via job submit with a batch inference spec
      When the user runs batch get with its id
      Then the CLI prints that job's state
    ```

- [ ] **FR-10**

    ```gherkin
    @FR-10
    Scenario: cancel a running job
      Given a batch job in a running state
      When the user runs batch cancel with its id
      Then the job reaches a cancelled state and no further logs are produced
    ```

- [ ] **FR-11**

    ```gherkin
    @FR-11
    Scenario: cancelling a finished job fails clearly
      Given a batch job in a succeeded state
      When the user runs batch cancel with its id
      Then the CLI exits non-zero with an error naming the job's succeeded state
    ```

- [ ] **FR-12**

    ```gherkin
    @FR-12
    Scenario: JSON output mode
      Given any command in the batch group that prints results
      When the user runs it with --json
      Then the CLI prints a single valid JSON document to stdout and nothing else
    ```

    ```gherkin
    @FR-12
    Scenario: JSON submission response
      Given a valid batch inference spec
      When the user runs batch submit <spec> --json
      Then the printed document contains the job id and nothing else is written to stdout
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable spec errors
      Given any batch spec rejected by local validation
      When the error is printed
      Then the message names the offending field, the likely cause, and one concrete fix
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given an API server that briefly drops the first connection
      When any batch command runs
      Then the CLI retries with backoff and succeeds once the server responds
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: spec secrets never leak
      Given a submitted spec containing a secret reference and env values
      When batch submit runs with verbose logging enabled and --json
      Then the secret's value and the env values appear nowhere in the output or log files
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: interactive speed on local paths
      Given a warm CLI cache
      When the user runs batch --help, or batch submit with an invalid spec
      Then the local part (startup, parse, validation, render) completes in under 500 ms
    ```

- [ ] **NFR-5**

    ```gherkin
    @NFR-5
    Scenario: cross-platform run
      Given Linux, macOS, and Windows hosts with the CLI installed
      When any batch command runs on each
      Then it behaves identically with no platform-specific code paths
    ```

- [ ] **NFR-6**

    ```gherkin
    @NFR-6
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors, and the batch group's argument surface is covered by pytest
    ```

- [ ] **NFR-7**

    ```gherkin
    @NFR-7
    Scenario: schema change propagates to both paths
      Given a change to the shared job spec schema or its local validation in the FEAT-p9-owned module
      When the test suites for job submit and batch submit run
      Then both exercise the changed behavior, with no forked copy of the schema or validator to update
    ```

## Conflicts

None identified yet.

## Open Questions

1. How is cursor pagination surfaced in `batch list` (flag names such as `--limit` and `--cursor`, or a transparent default page size)? The FEAT-p2 list endpoints paginate with cursors, but neither the seeded spec nor the FEAT-p1 command reference fixes the CLI flags.
2. Does `batch get` on a succeeded job surface the registered output dataset (name and location), or only the job state? The architecture registers the output as a dataset on success, but the seeded spec says only "current state".
3. What does `batch logs --follow` print when the job has already reached a terminal state before the stream opens: stored logs then exit, or an error?
4. Should `batch list` grow a `--name` filter (nightly scoring jobs are found by name), or is state filtering enough for v1?

Resolutions (2026-08-23): pagination is transparent cursor-following to exhaustion with no user-facing paging flags (FEAT-p9's landed model, settles 1); `batch get` renders the registered output dataset when present (settles 2); logs on an already-terminal job print stored lines then exit 0 (settles 3); the `--name` filter stays out of v1 (settles 4); server-confirmed unknown references exit 1 (settles 5); the idempotency key is fresh per invocation and reused across in-invocation retries (settles 6).
`--json --follow` on `batch logs` is a usage error matching `job logs`, so FR-12 holds exactly.

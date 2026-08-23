---
title: "mlx deploy command"
status: draft
parent: FEAT-p1
---

# Requirements: mlx deploy command

## Overview

MLEs need to serve registered models for online inference: put a model version behind an endpoint, watch it, change what it serves or how it scales, and release it when done.
This feature delivers the `mlx deploy` command group: deploy a registered model version, list deployments, inspect one, update the served model and scaling, and stop a deployment.
It implements FEAT-p1-FR-9 and FR-10 inside the unified CLI (FEAT-p1).
Requirement IDs stay owned by FEAT-p1; the local IDs in this document decompose those parent requirements for this command group and never redefine them.
No spec file is submitted to this group; the submitted input is the model reference and scaling flags.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Put a trained model behind an endpoint and keep it serving with minimal friction |
| ML infrastructure engineers | Scaling inputs validated before they reach the server, and errors that avoid support load |
| FEAT-p1 implementors | A complete deploy group that satisfies FEAT-p1-FR-9 and FR-10 |
| FEAT-p2 implementors | A consumer of the deployment endpoints whose contract expectations are explicit |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The `deploy create` command shall deploy a registered model version for online inference with optional scaling flags, generate the endpoint name when `--endpoint-name` is omitted, and report the endpoint reference (FEAT-p1-FR-9) |
| FR-2 | Must | The `deploy create` and `deploy update` commands shall validate scaling inputs locally before any server call: `--replicas` is mutually exclusive with `--min-replicas` and `--max-replicas`, the bound flags are passed together, min is less than or equal to max, and counts are positive integers; every failure exits non-zero naming the flag and the broken rule (FEAT-p1-FR-10, FEAT-p1-NFR-1) |
| FR-3 | Must | The `deploy create` and `deploy update` commands shall validate the model reference locally as `name:version` (kebab-case name, version per the registry convention), exiting non-zero with the expected form otherwise; existence of the model version is confirmed server-side with an error naming the unknown reference (FEAT-p1-FR-9, FR-7) |
| FR-4 | Must | The `deploy list` command shall list the deployments in the active workspace, each with its name, state, served model version, and endpoint reference (FEAT-p1-FR-10) |
| FR-5 | Must | The `deploy get <name>` command shall print one deployment's current state, served model version, scaling configuration (static count or autoscaling bounds), and endpoint reference, and shall exit non-zero with an error naming the deployment when no deployment with that name exists (FEAT-p1-FR-10) |
| FR-6 | Must | The `deploy update <name>` command shall accept any non-empty subset of `--model`, `--replicas`, `--min-replicas`, and `--max-replicas`; omitted flags keep their current values, setting `--replicas` clears autoscaling, and setting bounds replaces a static count; an empty subset is a usage error (FEAT-p1-FR-10) |
| FR-7 | Must | The `deploy stop <name>` command shall request that the server stop the deployment and release its resources, report the outcome, and exit non-zero with an error naming the deployment and its current state when it does not exist or is already stopped (FEAT-p1-FR-10) |
| FR-8 | Must | Updating the served model shall trigger a rollout whose progress is visible via `deploy get`, with the endpoint continuing to serve during the rollout (FEAT-p1-FR-10; `architecture.md` online inference flow) |
| FR-9 | Should | The `deploy create` command shall be safe to retry: a create retried after a transient failure (NFR-2) shall not create a second deployment |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every `deploy` subcommand shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix (inherits FEAT-p1-NFR-1) |
| NFR-2 | Must | Reliability | `deploy` server calls shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable (inherits FEAT-p1-NFR-2) |
| NFR-3 | Must | Performance | Interactive `deploy` subcommands (`list`, `get`) shall start up and render in under 500 ms on a warm cache (inherits FEAT-p1-NFR-4) |
| NFR-4 | Should | Maintainability | The `deploy` group shall have pytest coverage of its argument surface, and the codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors (inherits FEAT-p1-NFR-6) |

## Constraints

- This feature implements FEAT-p1-FR-9 and FR-10; requirement IDs stay owned by FEAT-p1, and local IDs in this document are subordinate decompositions.
- The command surface is fixed by the FEAT-p1 plan: `mlx deploy create|list|get|update|stop` with no additional subcommands or flags without a parent-feature change.
- No spec file is submitted to this group; the submitted input is the model reference and the scaling flags (input contract fixed by `spec.md`).
- Deployment state, rollout execution, endpoint provisioning, and resource release are server-side concerns owned by FEAT-p2; the CLI reports state and never derives it.
- The model registry surface (register, list, inspect) is owned by FEAT-p13; this group only references models as `name:version`.
- Python with cyclopts as the CLI framework; toolchain fixed by `infrastructure.md` (uv, ruff, pytest, ty).
- The CLI is a client of the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.
- Cross-cutting behavior (authentication, workspace scoping, JSON output mode, exit codes) is owned by FEAT-p1 (FR-2, FR-3, FR-12) and is not restated as local requirements.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: deploy a model version
      Given a registered model version and an authenticated user in a workspace
      When the user runs deploy create with the model reference
      Then the CLI reports the endpoint reference
      And the deployment appears in deploy list with a state
    ```

    ```gherkin
    @FR-1
    Scenario: endpoint name generated when omitted
      Given a registered model version
      When the user runs deploy create without --endpoint-name
      Then the reported endpoint reference carries a server-generated name
    ```

    ```gherkin
    @FR-1
    Scenario: explicit endpoint name is used
      Given a registered model version
      When the user runs deploy create with --endpoint-name fraud-scorer
      Then the reported endpoint reference carries fraud-scorer
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: static count combined with bounds is rejected
      Given a deploy create or deploy update invocation
      When the command passes --replicas together with --min-replicas or --max-replicas
      Then the CLI exits non-zero with an error naming --replicas and the mutual exclusion rule
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: lone bound flag is rejected
      Given a deploy create or deploy update invocation
      When the command passes --min-replicas without --max-replicas
      Then the CLI exits non-zero with an error naming the missing bound flag
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: min greater than max is rejected
      Given a deploy create or deploy update invocation
      When the command passes --min-replicas 8 --max-replicas 2
      Then the CLI exits non-zero with an error naming the min and max rule
      And no server call is made
    ```

    ```gherkin
    @FR-2
    Scenario: non-positive count is rejected
      Given a deploy create or deploy update invocation
      When the command passes --replicas 0
      Then the CLI exits non-zero with an error naming the positive-integer rule
      And no server call is made
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: malformed model reference is rejected locally
      Given the user runs deploy create or deploy update
      When the command passes ctr-estimator without a version
      Then the CLI exits non-zero with an error naming the expected name:version form
      And no server call is made
    ```

    ```gherkin
    @FR-3
    Scenario: unknown model version is named
      Given the registry holds no version 9 of model ctr-estimator
      When the user runs deploy create ctr-estimator:9
      Then the CLI exits non-zero and names the unknown model reference
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: list deployments
      Given the workspace contains two deployments in different states
      When the user runs deploy list
      Then every deployment appears with its name, state, served model version, and endpoint reference
    ```

    ```gherkin
    @FR-4
    Scenario: empty listing
      Given the workspace contains no deployments
      When the user runs deploy list
      Then the CLI exits zero with an empty listing
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: get one deployment
      Given one deployment serving a model version with a static replica count
      When the user runs deploy get with its name
      Then the CLI prints the deployment's state, served model version, replica count, and endpoint reference
    ```

    ```gherkin
    @FR-5
    Scenario: get a deployment that does not exist
      Given no deployment named fraud-scorer exists
      When the user runs deploy get fraud-scorer
      Then the CLI exits non-zero with an error naming fraud-scorer
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: update the served model
      Given one deployment serving ctr-estimator:1
      When the user runs deploy update with --model ctr-estimator:2
      Then deploy get shows ctr-estimator:2 as the served version
    ```

    ```gherkin
    @FR-6
    Scenario: omitted flags keep their values
      Given one deployment serving ctr-estimator:2 with a static replica count of 2
      When the user runs deploy update with only --replicas 3
      Then deploy get shows ctr-estimator:2 and 3 replicas
    ```

    ```gherkin
    @FR-6
    Scenario: set a static replica count
      Given one deployment with autoscaling enabled
      When the user runs deploy update with --replicas 3
      Then deploy get shows 3 replicas and no autoscaling bounds
    ```

    ```gherkin
    @FR-6
    Scenario: update autoscaling bounds
      Given one deployment with a static replica count of 1
      When the user runs deploy update with --min-replicas 2 --max-replicas 8
      Then deploy get shows the new bounds and no static count
    ```

    ```gherkin
    @FR-6
    Scenario: update with no flags
      Given one deployment
      When the user runs deploy update with its name and no flags
      Then the CLI exits non-zero with a usage error naming at least one required flag
      And no server call is made
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: stop a deployment
      Given one running deployment
      When the user runs deploy stop with its name
      Then the CLI reports the stop and deploy list shows the deployment stopped
    ```

    ```gherkin
    @FR-7
    Scenario: stop a deployment that does not exist
      Given no deployment named fraud-scorer exists
      When the user runs deploy stop fraud-scorer
      Then the CLI exits non-zero with an error naming fraud-scorer
    ```

    ```gherkin
    @FR-7
    Scenario: stop an already-stopped deployment
      Given one deployment in a stopped state
      When the user runs deploy stop with its name
      Then the CLI exits non-zero with an error naming the deployment and its current state
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: rollout progress is visible
      Given one deployment serving ctr-estimator:1
      When the user updates it to serve ctr-estimator:2
      Then a deploy get during the rollout reports the rollout in progress
      And a later deploy get reports ctr-estimator:2 serving
    ```

- [ ] **FR-9**

    ```gherkin
    @FR-9
    Scenario: retried create makes one deployment
      Given the API server briefly drops the first create request
      When the user runs deploy create and the CLI retries with backoff
      Then exactly one deployment is created and the CLI reports that deployment's endpoint reference
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable validation errors
      Given a deploy command with an invalid flag combination
      When the command fails on it
      Then the printed message names the likely cause and one concrete fix
    ```

    ```gherkin
    @NFR-1
    Scenario: help text present
      Given the CLI is installed
      When the user runs deploy --help and deploy create --help
      Then each prints usage with a description of every option
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the user runs deploy list
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs deploy list or deploy get
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
      And the deploy group's argument surface is covered by tests
    ```

## Conflicts

None identified yet.

## Open Questions

1. Should rollout progress stream during `deploy update`, or remain visible only via `deploy get`?
2. Should `deploy stop` require confirmation or a `--yes` flag?
3. Does `deploy create` return as soon as the server accepts the request, or wait for endpoint readiness?
4. What is the relationship between the deployment name accepted by `deploy get`, `deploy update`, and `deploy stop` and the endpoint reference printed by `deploy create` (one name, two names, or the endpoint reference alone)?
5. Which state values does the server report for deployments and rollouts in v1, and should the CLI render unknown state values verbatim (default) or reject them?
6. Is the version part of a model reference numeric (as the p9 spec schema assumes), or are arbitrary version strings allowed?

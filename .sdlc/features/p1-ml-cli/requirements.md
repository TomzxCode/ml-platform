---
title: "Unified ML CLI"
status: draft
---

# Requirements: Unified ML CLI

## Overview

MLEs need to run typical ML operations (data processing, data transfer, training, batch inference, online inference) at commercial scale without managing infrastructure themselves.
This feature delivers the CLI that lets them accomplish all of these operations through a single command surface, without requiring new commands per task.
The CLI is the client of the planned client-server architecture; the server, SDK, and database are out of scope for this feature.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Accomplish any typical ML operation from the CLI with minimal friction |
| ML infrastructure engineers | CLI behavior that avoids generating support load (clear errors, no footguns) |
| Leadership | First concrete step toward MLEs unencumbered by platform concerns |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The CLI shall expose commands covering every operation family: data processing, data transfer, training, batch inference, and online inference |
| FR-2 | Must | The CLI shall authenticate the user against the API server using a token before executing any operation |
| FR-3 | Must | The CLI shall let the user select a workspace and scope all subsequent operations to it |
| FR-4 | Must | The CLI shall submit data processing jobs defined by a user-supplied processing spec |
| FR-5 | Must | The CLI shall copy a dataset from a source location to a destination location on request |
| FR-6 | Must | The CLI shall submit training jobs defined by a user-supplied training spec (code, dataset reference, compute) |
| FR-7 | Must | The CLI shall let the user list registered models, inspect a model's versions and artifacts, and register an externally trained model artifact in the registry |
| FR-8 | Must | The CLI shall submit batch inference jobs that run a registered model over a dataset |
| FR-9 | Must | The CLI shall deploy a registered model for online inference and create an endpoint to serve it |
| FR-10 | Must | The CLI shall manage online inference deployments (list, inspect, update served model, static replica count and autoscaling bounds, stop) |
| FR-11 | Must | The CLI shall report job state for every submitted job (list jobs, inspect one job, stream its logs) |
| FR-12 | Should | The CLI shall emit machine-readable output (JSON) for every command when requested |
| FR-13 | Should | The CLI shall show transfer progress and resume an interrupted data copy |
| FR-14 | Should | The CLI shall cancel a submitted job on request |
| FR-15 | May | The CLI shall provide shell completion for supported shells |
| FR-16 | May | The CLI shall support named configuration profiles for multiple environments |
| FR-17 | Must | The CLI shall manage workspace secrets (set, list names, delete) that jobs reference for credentials, without ever displaying secret values |
| FR-18 | Should | The CLI shall list available compute types and inspect one type's resources and quota |
| FR-19 | Must | The CLI shall create experiments, list experiments in the workspace, and inspect one experiment's runs and their metrics |
| FR-20 | Must | The CLI shall manage dataset catalog entries (create from spec, list, inspect, delete) |
| FR-21 | Should | The CLI shall list resource quotas (compute, GPU, storage) with current utilization, and inspect one resource's quota |
| FR-22 | Should | The CLI shall create clusters from a spec, list clusters and their details, modify a cluster's configuration, and list machines across clusters with state and allocation |
| FR-23 | Should | The CLI shall list users with their team memberships and inspect one user's details |
| FR-24 | Should | The CLI shall manage workspace membership (list members with roles, add a member, remove a member) |
| FR-25 | Should | The CLI shall manage persistent volumes in the workspace (create, list, inspect, delete) |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Usability | Every command shall provide help text, and every failure shall print an actionable error message naming the likely cause and fix |
| NFR-2 | Must | Reliability | The CLI shall retry transient network failures with backoff and fail with a clear message when the API server is unreachable |
| NFR-3 | Must | Security | The CLI shall never write credentials to logs or non-interactive output |
| NFR-4 | Must | Performance | Interactive commands (help, list, status) shall start up and render in under 500 ms on a warm cache |
| NFR-5 | Should | Portability | The CLI shall install and run on Linux, macOS, and Windows via the standard Python toolchain, with no platform-specific code |
| NFR-6 | Should | Maintainability | The codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors, and every command group shall have pytest coverage of its argument surface |

## Constraints

- Python with cyclopts as the CLI framework.
- The CLI is a client of the planned API server (FastAPI/gRPC); the server itself is out of scope for this feature, so the CLI codes against the server's contract.
- Toolchain fixed by `infrastructure.md`: uv, ruff, pytest, ty.
- Compute dispatch (Kubernetes/cloud) happens server-side; the CLI never talks to compute infrastructure directly.
- Authentication is token-based.
- Distribution: installable with uv directly from the GitHub repository (`uv tool install git+<repo>`); no PyPI publication in v1.
- The CLI targets the API server designed in FEAT-p2; until that server exists, tests run against a contract-conformant stub.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: all operation families present
      Given the CLI is installed
      When the user runs the top-level help command
      Then the help lists commands covering data processing, data transfer, training, batch inference, and online inference
    ```

    ```gherkin
    @FR-1
    Scenario: unknown command
      Given the CLI is installed
      When the user runs a command that does not exist
      Then the CLI exits non-zero and suggests the closest matching command
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: first use requires authentication
      Given the user has never authenticated
      When the user runs any operation command
      Then the CLI prompts for an API token, authenticates against the API server, and proceeds on success
    ```

    ```gherkin
    @FR-2
    Scenario: authentication fails
      Given the user enters invalid credentials
      When the CLI attempts to authenticate
      Then the CLI exits non-zero with an authentication error and does not retry indefinitely
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: operations are workspace-scoped
      Given the user is authenticated and belongs to a team with two workspaces
      When the user selects workspace A and submits a training job
      Then the job is created in workspace A only
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: submit a data processing job
      Given a valid processing spec referencing an existing dataset
      When the user submits it via the job submission command
      Then the CLI accepts the spec, returns a job identifier, and the job appears in the job list
    ```

    ```gherkin
    @FR-4
    Scenario: invalid processing spec
      Given a processing spec missing a required field
      When the user submits it
      Then the CLI exits non-zero and names the missing field
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: copy a dataset
      Given the user has read access to a source dataset location and write access to a destination
      When the user requests a copy from source to destination
      Then the data appears at the destination and its contents match the source
    ```

    ```gherkin
    @FR-5
    Scenario: copy with inaccessible source
      Given a source location the user cannot read
      When the user requests a copy from it
      Then the CLI exits non-zero with a permissions error before transferring anything
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: submit a training job
      Given a valid training spec with code, a dataset reference, and a compute selection
      When the user submits it via the job submission command
      Then the CLI accepts the spec and returns a job identifier
    ```

    ```gherkin
    @FR-6
    Scenario: dataset reference does not exist
      Given a training spec referencing a dataset that does not exist
      When the user submits it
      Then the CLI exits non-zero and names the unknown dataset reference
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: list and inspect models
      Given the workspace contains at least one registered model with two versions
      When the user lists models and inspects one
      Then the listing shows the model and inspection shows both versions and their artifact locations
    ```

    ```gherkin
    @FR-7
    Scenario: register an external model
      Given a model artifact accessible to the user and not yet in the registry
      When the user registers it with a name
      Then the model appears in the registry with the artifact and a version
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: submit a batch inference job
      Given a registered model and a dataset in the workspace
      When the user submits a batch inference job spec via the batch submission command
      Then the CLI accepts the job and returns a job identifier
    ```

- [ ] **FR-9**

    ```gherkin
    @FR-9
    Scenario: deploy a model for online inference
      Given a registered model version
      When the user deploys it for online inference
      Then an endpoint is created for it and the CLI reports the endpoint reference
    ```

- [ ] **FR-10**

    ```gherkin
    @FR-10
    Scenario: manage deployments
      Given one running online inference deployment
      When the user lists deployments, inspects one, updates it, and stops it
      Then each command reflects the deployment's current state, ending with it stopped
    ```

    ```gherkin
    @FR-10
    Scenario: set a static replica count
      Given one running online inference deployment with autoscaling enabled
      When the user updates the deployment with a static replica count of 3
      Then inspecting the deployment shows 3 replicas and no autoscaling bounds
    ```

    ```gherkin
    @FR-10
    Scenario: update autoscaling bounds
      Given one running online inference deployment with min scale 1 and max scale 2
      When the user updates the deployment to min scale 2 and max scale 8
      Then inspecting the deployment shows the new bounds and an invalid range (min greater than max) is rejected with a clear error
    ```

- [ ] **FR-11**

    ```gherkin
    @FR-11
    Scenario: track a submitted job
      Given a submitted job in a running state
      When the user inspects it and then streams its logs
      Then the CLI shows the job's current state and appends log lines as they are produced
    ```

    ```gherkin
    @FR-11
    Scenario: job list
      Given the workspace contains jobs in different states
      When the user lists jobs
      Then every job appears with its identifier, type, and state
    ```

- [ ] **FR-12**

    ```gherkin
    @FR-12
    Scenario: JSON output mode
      Given any command that prints results
      When the user passes the machine-readable output flag
      Then the CLI prints a single valid JSON document to stdout and nothing else
    ```

- [ ] **FR-13**

    ```gherkin
    @FR-13
    Scenario: interrupted copy resumes
      Given a dataset copy interrupted midway
      When the user re-runs the same copy command
      Then the CLI resumes from where it stopped without re-copying completed parts
    ```

- [ ] **FR-14**

    ```gherkin
    @FR-14
    Scenario: cancel a job
      Given a submitted job in a running state
      When the user requests cancellation
      Then the job reaches a cancelled state and no further logs are produced
    ```

- [ ] **FR-15**

    ```gherkin
    @FR-15
    Scenario: generate shell completion
      Given a supported shell
      When the user runs the completion command
      Then a completion script for that shell is emitted
    ```

- [ ] **FR-16**

    ```gherkin
    @FR-16
    Scenario: switch configuration profiles
      Given two named configuration profiles pointing at different API servers
      When the user selects the second profile
      Then subsequent commands authenticate against and operate on the second server
    ```

- [ ] **FR-17**

    ```gherkin
    @FR-17
    Scenario: set and list a secret
      Given the user is authenticated in a workspace
      When the user sets a secret with a literal value and lists secrets
      Then the new secret's name appears in the listing and its value appears nowhere in the output
    ```

    ```gherkin
    @FR-17
    Scenario: delete a secret
      Given a secret exists in the workspace
      When the user deletes it and lists secrets again
      Then the secret no longer appears and jobs referencing it fail with a clear error naming the missing secret
    ```

- [ ] **FR-18**

    ```gherkin
    @FR-18
    Scenario: discover compute types
      Given the workspace has compute quota
      When the user lists compute types and inspects one
      Then the listing shows the available types and the inspection shows GPUs, memory, and quota for that type
    ```

- [ ] **FR-19**

    ```gherkin
    @FR-19
    Scenario: create an experiment
      Given the user is authenticated in a workspace
      When the user creates an experiment from a valid spec
      Then the experiment appears in the workspace's experiment list
    ```

    ```gherkin
    @FR-19
    Scenario: inspect an experiment
      Given the workspace contains an experiment with at least two runs
      When the user lists experiments and inspects one
      Then the listing shows the experiment and the inspection shows each run with its metrics
    ```

- [ ] **FR-20**

    ```gherkin
    @FR-20
    Scenario: create and inspect a dataset entry
      Given a valid dataset spec pointing at an accessible location
      When the user creates a dataset from it and gets the dataset
      Then the dataset appears in the listing with its location, format, and version
    ```

    ```gherkin
    @FR-20
    Scenario: delete a dataset entry
      Given a dataset exists in the catalog
      When the user deletes it and lists datasets again
      Then the dataset no longer appears and jobs referencing it fail with a clear error naming the missing dataset
    ```

- [ ] **FR-21**

    ```gherkin
    @FR-21
    Scenario: inspect quotas
      Given the workspace has compute, GPU, and storage quotas with some utilization
      When the user lists quotas and gets one resource
      Then the listing shows each resource with used and total, and the inspection shows that resource's utilization in detail
    ```

- [ ] **FR-22**

    ```gherkin
    @FR-22
    Scenario: create a cluster
      Given a valid cluster spec with cloud, region, and machine types
      When the user creates a cluster from it
      Then the cluster appears in the cluster list in a provisioning state and later reaches a healthy state
    ```

    ```gherkin
    @FR-22
    Scenario: update a cluster
      Given a healthy cluster without a GPU machine type
      When the user updates the cluster to add that machine type and raises the autoscaler ceiling
      Then inspecting the cluster shows the new machine type and ceiling
    ```

    ```gherkin
    @FR-22
    Scenario: browse infrastructure
      Given the platform runs on at least two clusters with machines in various states
      When the user lists clusters, inspects one, and lists machines filtered to it
      Then the cluster listing shows both clusters with health, the inspection shows its details, and the machine listing shows only that cluster's machines with state and allocation
    ```

- [ ] **FR-23**

    ```gherkin
    @FR-23
    Scenario: browse users
      Given the caller has permission to list users
      When the user lists users and inspects one
      Then the listing shows users with team memberships and the inspection shows identity, teams, and workspaces
    ```

- [ ] **FR-24**

    ```gherkin
    @FR-24
    Scenario: manage workspace membership
      Given the caller is an admin of the active workspace and a second user is not a member
      When the user adds the second user with a role, lists members, and removes them again
      Then the member listing shows the addition with role and then shows the removal
    ```

    ```gherkin
    @FR-24
    Scenario: non-admin cannot modify membership
      Given the caller is a non-admin member of the active workspace
      When the user attempts to add a member
      Then the CLI exits non-zero with a permissions error
    ```

- [ ] **FR-25**

    ```gherkin
    @FR-25
    Scenario: manage volumes
      Given the user is authenticated in a workspace with volume quota available
      When the user creates a volume, lists volumes, inspects it, and deletes it
      Then the volume appears in the listing, inspection shows its details, and deletion removes it
    ```

    ```gherkin
    @FR-25
    Scenario: quota exceeded
      Given the workspace's storage quota is exhausted
      When the user attempts to create a volume
      Then the CLI exits non-zero with a quota error naming the exhausted resource
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: actionable errors
      Given a command that fails due to user input
      When the error is printed
      Then the message names the likely cause and one concrete fix
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: transient network failure is retried
      Given the API server briefly drops the first connection
      When the CLI makes a request
      Then the CLI retries with backoff and succeeds once the server responds
      And the user sees at most one informational line about the retry
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: credentials never leak
      Given an authenticated session
      When any command runs with verbose logging enabled
      Then the credentials or token appear nowhere in the output or log files
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: interactive command speed
      Given a warm CLI cache
      When the user runs a help, list, or status command
      Then it renders its output in under 500 ms
    ```

- [ ] **NFR-5**

    ```gherkin
    @NFR-5
    Scenario: installs on Linux and macOS
      Given a Linux host and a macOS host with the standard Python toolchain
      When the CLI is installed via uv on each
      Then installation succeeds and `--help` works on both
    ```

- [ ] **NFR-6**

    ```gherkin
    @NFR-6
    Scenario: toolchain gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
    ```

## Conflicts

None identified yet.

## Open Questions

None.

Resolutions (2026-08-23): token-based auth (FR-2); distribution via uv from the GitHub repository (Constraints); Windows supported with nothing platform-specific (NFR-5); the CLI talks to the API server designed as FEAT-p2 (Constraints).

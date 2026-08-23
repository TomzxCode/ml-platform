---
title: "API Server"
status: draft
---

# Requirements: API Server

## Overview

The CLI (FEAT-p1) needs a server to talk to.
This feature delivers the API server: the central component that processes and dispatches user requests to the appropriate infrastructure and makes placement decisions.
It defines and implements the API contract (REST via FastAPI, documented in OpenAPI 3, plus gRPC services) covering all operation families (data processing, data transfer, training, batch inference, online inference), backed by PostgreSQL and deployed on Kubernetes.

## Stakeholders

| Stakeholder | Interest |
|---|---|
| MLEs | Reliable execution of their operations submitted through the CLI |
| ML infrastructure engineers | Operable server (k8s deployment, clear logs, low support load) |
| CLI implementors (FEAT-p1) | A stable, documented contract to code against |

## Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Requirement |
|---|---|---|
| FR-1 | Must | The server shall expose a documented API contract (OpenAPI 3 for REST, proto definitions for gRPC) covering data processing, data transfer, training, batch inference, and online inference |
| FR-2 | Must | The server shall authenticate every request with a token and reject invalid or expired tokens |
| FR-3 | Must | The server shall manage teams, users, and workspaces, and enforce that every resource belongs to exactly one workspace |
| FR-4 | Must | The server shall accept data processing job submissions and return a job identifier |
| FR-5 | Must | The server shall accept data transfer requests (source to destination) and execute the copy |
| FR-6 | Must | The server shall accept training job submissions (code, dataset reference, compute selection), track checkpoints, and register the final model version in the model registry |
| FR-7 | Must | The server shall provide a model registry: register versions and artifacts, list models, inspect a version |
| FR-8 | Must | The server shall accept batch inference jobs over a registered model and dataset |
| FR-9 | Must | The server shall create and manage online inference deployments and their endpoints (inspect, update, stop) |
| FR-10 | Must | The server shall report job state for every job and stream job logs to clients |
| FR-11 | Must | The server shall make placement decisions and dispatch work to Kubernetes and cloud APIs |
| FR-12 | Must | The server shall persist all durable state in PostgreSQL |
| FR-13 | Should | The server shall manage token lifecycle (issue, revoke, expire) |
| FR-14 | Should | The server shall paginate and filter all list endpoints |
| FR-15 | May | The server shall emit state-change events (webhooks or event stream) for jobs and deployments |

## Non-Functional Requirements

Order rows by priority: Must first, then Should, then May.

| ID | Priority | Category | Requirement |
|---|---|---|---|
| NFR-1 | Must | Security | The server shall store tokens hashed, and shall never write tokens to logs or error output |
| NFR-2 | Must | Reliability | The server shall survive restarts with no loss of submitted-job state (stateless pods, durable state in PostgreSQL) |
| NFR-3 | Must | Observability | The server shall emit structured logs with a request identifier on every request |
| NFR-4 | Should | Performance | The server shall acknowledge job submissions in under 1 second at p95 (excluding queue wait) |
| NFR-5 | Should | Maintainability | The codebase shall pass `ruff check`, `ruff format --check`, and `ty check` with no errors, and the API contract shall have pytest coverage against a contract test suite |

## Constraints

- Python with FastAPI (REST) and gRPC services.
- PostgreSQL for persistence.
- Deployed on Kubernetes instances.
- Token-based authentication (per FEAT-p1 resolution).
- The CLI (FEAT-p1) is the first contract consumer; the contract must satisfy its requirements.
- CI via GitHub Actions.

## Acceptance Criteria

Every FR and NFR shall have at least one acceptance criterion.

Order criteria by FRs first (sorted by ID), then NFRs (sorted by ID).

- [ ] **FR-1**

    ```gherkin
    @FR-1
    Scenario: contract covers all operation families
      Given the running server
      When the OpenAPI document and proto definitions are retrieved
      Then they cover resources and operations for data processing, data transfer, training, batch inference, and online inference
    ```

- [ ] **FR-2**

    ```gherkin
    @FR-2
    Scenario: valid token is accepted
      Given a user with a valid token
      When they call any authenticated endpoint with the token
      Then the request is processed
    ```

    ```gherkin
    @FR-2
    Scenario: invalid token is rejected
      Given a request with an invalid or expired token
      When it reaches the server
      Then the server responds with an authentication error and performs no operation
    ```

- [ ] **FR-3**

    ```gherkin
    @FR-3
    Scenario: resources are workspace-scoped
      Given a user with access to workspace A only
      When they request a resource in workspace B
      Then the server denies the request
    ```

- [ ] **FR-4**

    ```gherkin
    @FR-4
    Scenario: submit a data processing job
      Given an authenticated user with a valid processing spec
      When they submit it
      Then the server returns a job identifier and the job appears with a state
    ```

- [ ] **FR-5**

    ```gherkin
    @FR-5
    Scenario: execute a data transfer
      Given an authenticated user with read access to a source and write access to a destination
      When they request a transfer
      Then the server executes the copy and reports completion
    ```

- [ ] **FR-6**

    ```gherkin
    @FR-6
    Scenario: training job registers its final model
      Given a submitted training job that runs to completion
      When the job finishes
      Then the final checkpoint is registered as a new model version in the model registry
    ```

- [ ] **FR-7**

    ```gherkin
    @FR-7
    Scenario: inspect a model version
      Given a registered model with at least one version
      When a user lists models and inspects a version
      Then the response includes the version's artifact location and lineage (which job produced it)
    ```

- [ ] **FR-8**

    ```gherkin
    @FR-8
    Scenario: submit a batch inference job
      Given a registered model version and an accessible dataset
      When a user submits a batch inference job over them
      Then the server returns a job identifier and the job executes to completion
    ```

- [ ] **FR-9**

    ```gherkin
    @FR-9
    Scenario: manage an online inference deployment
      Given a registered model version
      When a user creates a deployment, updates it, and stops it
      Then the endpoint serves the model between create and stop, and stops serving after stop
    ```

- [ ] **FR-10**

    ```gherkin
    @FR-10
    Scenario: stream job logs
      Given a running job producing logs
      When a client subscribes to the job's logs
      Then the server streams log lines as they are produced
    ```

- [ ] **FR-11**

    ```gherkin
    @FR-11
    Scenario: dispatch to infrastructure
      Given a job ready to run and available compute
      When the scheduler places it
      Then the job is dispatched to Kubernetes or the cloud API and starts running
    ```

- [ ] **FR-12**

    ```gherkin
    @FR-12
    Scenario: state survives a restart
      Given a server with submitted jobs
      When the server process restarts
      Then all jobs and their states are unchanged
    ```

- [ ] **FR-13**

    ```gherkin
    @FR-13
    Scenario: revoke a token
      Given an administrator and an active user token
      When the administrator revokes it
      Then subsequent requests with that token are rejected
    ```

- [ ] **FR-14**

    ```gherkin
    @FR-14
    Scenario: paginate a list endpoint
      Given a workspace with more jobs than the page size
      When a client requests a page
      Then the response contains at most the page size and a cursor or offset for the next page
    ```

- [ ] **FR-15**

    ```gherkin
    @FR-15
    Scenario: job state change event
      Given a client subscribed to events
      When a job changes state
      Then the client receives an event naming the job and its new state
    ```

- [ ] **NFR-1**

    ```gherkin
    @NFR-1
    Scenario: tokens are never logged
      Given a server processing authenticated requests with debug logging enabled
      When logs are inspected
      Then no token value appears anywhere
    ```

- [ ] **NFR-2**

    ```gherkin
    @NFR-2
    Scenario: pod restart loses nothing
      Given jobs in running and queued states
      When an API pod is killed and replaced
      Then queued jobs are still queued, running jobs are reconciled, and no submission is lost
    ```

- [ ] **NFR-3**

    ```gherkin
    @NFR-3
    Scenario: request identifiers in logs
      Given any incoming request
      When the server logs it
      Then the log line carries a request identifier that also appears in the response
    ```

- [ ] **NFR-4**

    ```gherkin
    @NFR-4
    Scenario: submission latency
      Given a healthy server under normal load
      When a job submission is measured
      Then it is acknowledged in under 1 second at p95
    ```

- [ ] **NFR-5**

    ```gherkin
    @NFR-5
    Scenario: toolchain and contract gates pass
      Given the repository
      When `uv run ruff check .`, `uv run ruff format --check .`, `uv run ty check .`, and `uv run pytest` run
      Then all four pass with no errors
    ```

## Conflicts

None identified yet.

## Open Questions

1. Which artifact/checkpoint storage backend (S3, GCS, MinIO, other)?
2. Who issues tokens in v1 (a login endpoint, an admin command, or an external identity provider)?
3. Which cloud provider(s) must the placement/dispatch layer target in v1?
4. Are REST and gRPC both exposed in v1, or REST first with gRPC reserved for log streaming?
5. What idempotency semantics do submission endpoints guarantee on client retry?

# Vocabulary

## Domain Terms

| Term | Definition |
|---|---|
| Team | Group of users |
| User | Person interacting with the platform |
| Workspace | A way to isolate resources (secrets, datasets, access, etc.) between teams/users, similar to Kubernetes namespaces |
| Workspace membership | A user's membership in a workspace, with a role |
| Workspace role | `admin` or `member`; the workspace creator becomes its first admin |
| Dataset | Cataloged data source usable by jobs |
| Model | Registered model; a family of versions |
| Model version | Versioned model artifact with a location and lineage to the producing job; versions are positive integers, newest first, defaulting to the next number under the name |
| Endpoint | Online inference deployment target |
| Experiment | Tracked training run grouping |
| Run | Tracked run within an experiment, with metrics |
| Data processing job | Executes data processing on compute |
| Training job | Executes training on compute |
| Batch inference job | Executes batch inference on compute |
| Online inference deployment | Serves a model for online inference; states: creating, serving, updating, stopped, failed |
| Rollout | In-flight update of a deployment's served model version; the endpoint keeps serving until it completes |
| Secret | Workspace-scoped credential referenced by jobs for authentication to external systems |
| Compute type | Machine type with GPUs and memory, available to a workspace under quota |
| Resource quota | Limit on a resource (compute, GPU, storage) set on a scope (workspace or team), with current utilization |
| Persistent volume | Workspace-scoped storage volume for staging data and checkpoints |
| Transfer | Server-executed data copy from a source to a destination location, with progress and resume |
| Cluster | Kubernetes cluster the platform runs on, with region and health |
| Machine | Machine within a cluster, with state and allocation |
| API key | Named credential for programmatic access (CI, automation), owned by a user or service account; its value is displayed only once at creation |
| Service account | Non-human identity scoped to a workspace, for machine access to the platform |
| Job spec | YAML file defining a job, discriminated by `type` (processing, training, batch-inference) |

## Technical Terms

| Term | Definition |
|---|---|
| CLI | Command-line interface; the user-facing command surface |
| mlx | The CLI binary name |
| CLI profile | Named configuration (server URL, credentials, selected workspace) stored under the platform config directory |
| API server | Processes and dispatches user requests to the appropriate infrastructure; makes placement decisions |
| API contract | Versioned OpenAPI 3 document plus gRPC protos shared by the CLI and the server |
| Contract-conformant stub | Fake server implementing the published contract, used by CLI tests until the real server exists |
| Job scheduler | Schedules training and batch inference jobs on compute |
| Placement | Server-side decision of where a job runs, made against workspace compute quota |
| Dispatch | Server-side execution of accepted work on Kubernetes |
| Reconciliation | Matching Kubernetes state to database state on server startup |
| Model registry | Stores and versions model checkpoints/artifacts |
| Dataset catalog | Catalogs and indexes datasets |
| Data transfer tools | Copy data from source A to destination B |
| Lineage | Traceability from a model version to the job that produced it |
| Idempotency key | Header on submission endpoints that makes client retries safe via replay semantics |
| Cursor pagination | List-endpoint convention returning a cursor for the next page |
| Request id | Identifier stamped on every server request, present in logs and echoed in responses |
| ComputeBackend | Server-side interface abstracting compute targets (Kubernetes in v1; an in-process fake for development and tests) |
| ArtifactStorage | Server-side interface abstracting artifact storage (S3-compatible; MinIO in development) |
| CLI telemetry | Anonymous, opt-out usage events (`source: cli`, `install_id`, `cli_version`) used to measure onboarding and command health; never carries tokens, identity, or server URLs |
| Install id | Anonymous per-install identifier created at first CLI run, keyed for telemetry funnels |
| Output format | Rendering of list and get commands: `json`, `yaml`, `table` (default), or `name` (one name per line for shell piping) |
| Exit code | CLI process result: 0 success, 1 usage error, 2 authentication failure, 3 server error, 4 cancelled by user |

## Acronyms

| Acronym | Expansion |
|---|---|
| ML | Machine learning |
| MLE | Machine learning engineer |
| GHA | GitHub Actions |
| k8s | Kubernetes |
| REST | Representational State Transfer |
| SSE | Server-Sent Events (REST log streaming) |

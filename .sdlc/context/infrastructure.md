# Infrastructure

## Technology Stack

| Layer | Choice |
|---|---|
| Language | Python >= 3.14 (CLI and server) |
| CLI framework | cyclopts |
| Server framework | FastAPI + gRPC |
| API contract | OpenAPI 3 (REST) plus protobuf (gRPC log streaming) |
| Database | PostgreSQL |
| Migrations | alembic |
| Logging | structlog (structured logs with request identifiers) |
| Artifact storage | S3-compatible (MinIO in development) |
| Orchestration | Kubernetes |
| Documentation | MkDocs Material (`docs/`, `docs` dependency group) |

## Development Tooling

| Tool | Purpose | Command |
|---|---|---|
| uv | Package management | `uv sync`, `uv add <pkg>` |
| ruff | Linting and formatting | `uv run ruff check .`, `uv run ruff format .` |
| pytest | Testing | `uv run pytest` |
| ty | Type checking | `uv run ty check .` |
| mkdocs | Documentation site | `uv run mkdocs serve`, `uv run mkdocs build` |

The repository is scaffolded (`pyproject.toml`, version 0.1.0, `uv.lock`); source packages are not yet present.

Planned additions once code lands (per the FEAT-p1 and FEAT-p2 plans): alembic for database migrations (`uv run alembic upgrade head`), a contract test suite against the published OpenAPI document, and kind (Kubernetes-in-Docker) for dispatch integration tests in CI.

## Environments

- Local development: standard Python workflow; the server runs against the in-process fake ComputeBackend and MinIO for artifacts.
- Integration testing: kind cluster in GHA (planned).
- Server: deployed on Kubernetes as stateless pods, durable state in PostgreSQL.

## Distribution

- The CLI installs with uv directly from the GitHub repository (`uv tool install git+<repo>`).
- No PyPI publication in v1.

## CI/CD

- None yet.
- Plan: GHA running the four gates (`ruff check`, `ruff format --check`, `ty check`, `pytest`) for both packages, plus the contract test suite and kind-based integration tests (FEAT-p1 Phase 1; FEAT-p2 api Phase 6).

## Deployment and Rollback

- Placeholder: k8s deployment manifests for stateless server pods, with a documented rollout and rollback procedure, are a FEAT-p2 deliverable (api Phase 6).

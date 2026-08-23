# Infrastructure

## Technology Stack

| Layer | Choice |
|---|---|
| Language | Python (CLI and server) |
| CLI framework | cyclopts |
| Server framework | FastAPI + gRPC |
| Database | PostgreSQL |

## Development Tooling

| Tool | Purpose | Command |
|---|---|---|
| uv | Package management | `uv sync`, `uv add <pkg>` |
| ruff | Linting and formatting | `uv run ruff check .`, `uv run ruff format .` |
| pytest | Testing | `uv run pytest` |
| ty | Type checking | `uv run ty check .` |

## Environments

- Server: deployed on Kubernetes instances.
- Local development: standard Python workflow.

## CI/CD

- None yet.
- Plan: GitHub Actions (GHA).

## Deployment and Rollback

- Placeholder: deployment and rollback procedures to be defined when CI/CD and environments are set up.

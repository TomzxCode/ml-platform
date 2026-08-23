# Project Overview

## Purpose

ml-platform is responsible for all things machine learning: it provides a unified interface for users to do ML at commercial scale.
That includes processing data, training models, running batch inference, deploying models for online inference, and copying data from A to B.
The goal is a single surface covering anything relevant to doing machine learning at scale commercially.

## Key Stakeholders

| Stakeholder | Role | Interest |
|---|---|---|
| MLEs in companies | Primary users | Least amount of friction with the infrastructure they need to get their work done |
| ML infrastructure engineers | Platform maintainers | As little on-call load and as few support requests as possible |
| Leadership | Sponsors | MLEs unencumbered by platform concerns |

## Scope

**In scope:**
- The unified CLI (FEAT-p1): the `mlx` command surface abstracting all ML operations (data processing, data transfer, training, batch inference, online inference deployment).
- The API server (FEAT-p2): the OpenAPI 3 contract, the FastAPI/gRPC server, PostgreSQL persistence, and Kubernetes dispatch that back every CLI operation.
- Both packages are developed in this repository with a src layout.

**Out of scope:**
- SDK.
- PyPI publication in v1 (the CLI installs from the GitHub repository via uv).
- SSO and mTLS authentication in v1 (token-based only).
- Dispatch targets other than Kubernetes in v1.

## Key Constraints

- Authentication is token-based in v1.
- Kubernetes is the only compute dispatch target in v1.
- The CLI is distributed via uv from the GitHub repository.
- The CLI is a client; compute dispatch happens server-side, and the CLI never talks to compute infrastructure directly.

## Current Status

The repository is scaffolded (`pyproject.toml`, `uv.lock`) and carries a MkDocs documentation site (`docs/`, `mkdocs.yml`); no source packages exist yet.
Planning artifacts live under `.sdlc/features/`: FEAT-p1 (CLI requirements and plan), FEAT-p2 (server requirements and a four-concern plan), and child features FEAT-p3 to FEAT-p14 decomposing the CLI command groups, most of which now have reviewed specifications and plans; telemetry plans exist for the auth, compute, job, batch, deploy, and model groups.

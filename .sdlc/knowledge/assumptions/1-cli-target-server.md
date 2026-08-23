---
status: Active
---

# Assumption: A usable API server exists (or will exist) for the CLI to target in v1

**Date:** 2026-08-23
**Status:** Active
**Author:** ml-platform implementors

---

## Statement

The unified ML CLI (FEAT-p1) can be built and its acceptance criteria exercised against some runnable form of the planned API server (a real deployment, a reference implementation, or a contract-conformant stub) even though the server itself is out of scope for this feature.

## Basis

The project overview lists Server/SDK/DB as out of scope for the first release, while the architecture (client-server, FastAPI/gRPC) is the stated direction.
No server implementation is known to exist today.

## Confidence

**Level:** Low

The scoping decision excludes the server but does not state what the CLI integrates and tests against.

## Risk if Wrong

**Impact:** High

Every Must requirement (FR-2 through FR-11) exercises the API server contract.
With no server or stub, end-to-end acceptance criteria cannot be executed, and the feature reduces to untestable scaffolding or its scope must shrink.

## Validation Plan

**Method:** Answer FEAT-p1 requirements Open Question 1 before specifications begin: decide between (a) an existing/reference server, (b) coding against the documented contract with a contract-conformant stub for tests, or (c) descoping.
**Target Date:** 2026-09-06

## Update

2026-08-23: Resolved by decision, option (b): the API server will be designed in-house as FEAT-p2; the CLI codes against that contract with a contract-conformant stub for tests until the server exists.
Formal validation pending `/review-assumption`.

## Outcome

---
artifact: specification
verdict: approved
reviewed_at: 2026-08-23
---

## Ambiguities

No issues found.
Field names, types, constraints, the flag-to-scaling-object mapping, validation order, error mapping, pagination, and rollout rendering are explicit.
The author's pre-handoff self-check extended the tolerance rule to unknown `scaling.mode` values before this review.
Deliberate openness (not ambiguity): the endpoint reference's format is unspecified beyond "server-reported, rendered verbatim", because its shape (bare name versus URL) is a contract concern; the name is the addressing key and the one-name design carries the resolution risk (Open Questions 4 and 7).

## Inconsistencies

No blocking findings.
No `api.yaml` exists and none is required: the feature defines no API surface, and the consumer-side table cites the normative owner (FEAT-p2 contract) with an explicit drift rule (FEAT-p2 wins), matching the approved sibling posture.
Data models match the contract table and the sequences: create (201), unknown reference (400 MODEL_NOT_FOUND), list (200 with cursor), get (200 with rollout when in flight), update (200 with rollout), stop (200 stopped), and the 401/404/409/5xx rows of the error mapping.
No orphan operations: every path in the summary table appears in a sequence or the error mapping, and vice versa.
The cli-design JSON shapes match the resource model defined here (summaries as `name`/`state`/`model`/`endpoint`; create, update, and get emitting the deployment object; stop emitting `name`/`state`).
Environment note: the mermaid render check could not run in this environment (the bundled Chromium cannot start under the AppArmor sandbox restriction); all nine blocks use the construct set of the approved p9 specification (flowchart LR, sequenceDiagram with autonumber, notes, one `<br/>` in a note), and the one risky construct found in self-check (a semicolon inside a note) was removed before this review.

## Incoherences

No issues found.
Technical decisions do not contradict each other: create returns on acceptance while rollout progress is observed via `deploy get`, which keeps the CLI decoupled from server-side rollout timing; the one-name design coheres with the endpoint being rendered verbatim (the name addresses, the endpoint is displayed); the closed local model-reference form versus the open-on-read `state` enum mirrors the p9 asymmetry with the same rationale (input the CLI validates versus state the server owns).
The architecture adds no layers and stays inside the stated constraints (client only, server authority).

## Missing Information

No blocking findings.
Every FR and NFR from `requirements.md` is addressed: FR-1 via the create sequence and the endpoint-name generation rule, FR-2 via the scaling input table and deterministic validation order, FR-3 via the local form check plus the 400 MODEL_NOT_FOUND row, FR-4 via the list sequence and summaries, FR-5 via the get sequence, resource fields, and the 404 row, FR-6 via the update semantics (omitted fields, static versus bounds replacement, empty-update rejection), FR-7 via the stop sequence and the 409 row, FR-8 via the rollout rendering rule and update sequence, FR-9 via the idempotency sequence, NFR-1/NFR-2 via the inherited error model and retry mapping, NFR-3 via the interactive-budget decision, NFR-4 via the testing-posture decision.
Authentication, workspace scoping, and the error model are inherited from FEAT-p2 and stated once.
Observations:

- The default scaling (static 1) and the empty-update rejection, flagged at requirements review as cli-design-only, are now carried into the scaling input table and the validation order, closing the p8/p9-style gap.
- Empty-result behavior for `deploy list` (exit 0, empty table or empty JSON array) is stated in cli-design.md; the specification's "complete array" wording implies the empty array but does not spell it out, the same residual wording as the approved p9 specification.
- The `model` field during a rollout is defined as the target version (get sequence note), which is the only interpretation rule for the resource's model field; it is stated once and matches the rollout object's `to_model`.

## Implementability

No issues found.
All choices stay within cyclopts plus the shared client; no new infrastructure is required.
The stub extension needed for testing (deployment endpoints with rollout behavior, idempotent create replay, and the 409 paths) is a plan-level deliverable, and risk 5 records the no-server posture with parent assumption 1.
External dependencies are explicit: the FEAT-p2 deployment endpoints and the three consumer-side error-code requests (risk 2).

## Reversibility

No issues found.
The feature holds no migrations, deployments, or persisted state; every change is code in the parent CLI package, revertible by commit.
The recorded defaults (return on acceptance, get-only rollout visibility, one-name design, no stop prompt, empty-update rejection) are each isolated behind a single handler or decision row, so reversing any of them is local.
The one-way-door risk is acknowledged rather than hidden: if FEAT-p2 assigns distinct deployment and endpoint names, the change is confined to create's output and one name-mapping point (risk 3).

## Forward Compatibility

No issues found.
Unknown response fields are tolerated; unknown `state`, `rollout.status`, and `scaling.mode` values render verbatim; the name and endpoint are treated as opaque server strings; the additive-only compatibility policy is inherited from the FEAT-p2 contract conventions.
Conscious tradeoffs (not gaps): the local model-reference regex is closed in v1 (a non-numeric registry version relaxes one regex, risk 4), and the scaling request object the CLI constructs always uses the two v1 modes, which is an input the server validates anyway.

## Open Questions

Carried from `requirements.md` and `review-requirements.md`, each with a recorded default in Technical Decisions where one was needed:

1. Rollout progress streaming during `deploy update` (default: `get` only).
2. `deploy stop` confirmation or `--yes` (default: no prompt).
3. Create returns on acceptance versus waiting for readiness (default: return on acceptance).
4. Deployment name versus endpoint reference identity (default: one name).
5. Server-reported state vocabulary (default: open on read, illustrative values).
6. Numeric versus arbitrary model version (default: numeric).
7. Default scaling when every flag is omitted (default: static 1).

New from this specification:

8. FEAT-p2 must publish the deployment schemas, the rollout representation, and the `MODEL_NOT_FOUND`, `ENDPOINT_NAME_TAKEN`, and stop-refusal `INVALID_STATE` codes this consumer view assumes (risks 1 and 2).

Note: per this run's write scope (feature directory only), no assumption or decision records were created under `.sdlc/knowledge/`.
Questions 4, 5, and 8 carry the most implementation risk and are the first candidates for formal records when that path is writable.

---
artifact: lifecycle
verdict: approved
reviewed_at: 2026-08-23
---

## Completeness

No blocking findings.
The one managed-and-tracked resource (Deployment) has a state diagram, per-state entry and exit conditions, transitions with trigger, actor, side effects, and guards, six invariants with enforcement and violation handling, and a retention row.
The author's pre-handoff self-check recorded the scaling input as a per-invocation request field with no lifecycle entry, the scaling-only update as an explicit state-preserving transition, and the command-level telemetry events as distinct from the transition-tied ones.
Observations: the Deployment is a client view of a server-owned state machine; the ownership boundary is stated in the Overview and repeated in Out of Scope, which keeps this document from drifting into FEAT-p2 territory, the same posture as the approved p9 job lifecycle.

## Consistency

No issues found.
State names match the specification's illustrative v1 enum exactly (`creating`, `serving`, `updating`, `stopped`, `failed`) under the shared open-on-read rule.
The create, update, and stop transitions correspond to the POST, PATCH, and stop endpoints with the 404 and 409 rows of the specification's error mapping; the create transition corresponds to the 201 in the create sequence.
Event names (`deploy_create_succeeded`, `deploy_create_failed`, `deploy_update_succeeded`, `deploy_update_failed`, `deploy_stop_succeeded`, `deploy_stop_failed`, `deploy_state_reported`) forward-reference telemetry.md, which is written in the same run with the same taxonomy.

## Spec Alignment

No issues found.
Every user-triggered transition traces to a specification endpoint or sequence: create to the happy-path and retry sequences, update to the rollout sequence, stop to the stop sequence, the read commands to the list and get sequences.
Provisioning and rollout transitions are marked "Observed only", matching the constraint that the CLI never derives state.
Guard conditions restate the specification's local validation rules (forms, combination, bounds pairing) and defer terminal-state adjudication to the server.
The author's self-check tightened two guard and side-effect wordings before this review so no claim exceeds what the specification's error mapping supports (update-on-terminal and rollout-failure disposition are both server-adjudicated).

## Transition Correctness

No issues found.
Every state is reachable (Creating via create; Serving via provisioning or rollout completion; Updating via a model update; Stopped via stop; Failed via provisioning or rollout failure) and Stopped and Failed are dead ends by design.
The backward transition (Updating to Serving) is documented with its server trigger, and the three races (stop versus rollout, stop during Creating, update racing provisioning) are documented with the server as the adjudicator.
The transitions the server may perform but the CLI does not model (for example Serving to Failed on an infrastructure error, or an update accepted during Creating) are covered by the open-on-read note under the diagram, so they are documented gaps, not missing ones.

## Invariant Soundness

No issues found.
Each invariant holds in every state: read-only rendering (including unknown states), mutation only through the three endpoints, scaling construction only through the single validation module, one idempotency key per create invocation, rollout rendered from the server-reported object only, and name opacity after creation.
Enforcement mechanisms are specific (read-only handlers, the shared validation module, client-core key handling, output and retry tests) and violation handling is realistic (test or review failure, matching the repository's gates).

## Retention Soundness

No issues found.
The client retains nothing between invocations, so the client-side retention row is a deliberate no-op with the expiry trigger (process exit) and cleanup action (nothing) stated.
Retention of stopped deployment records is explicitly deferred to FEAT-p2 in the row and in Out of Scope, which is the correct boundary for a client feature; if the server retains stopped records indefinitely, that policy is the server owner's to publish.

## Open Questions

Carried from the requirements and specification reviews; none are new to this artifact:

1. Deployment name versus endpoint reference identity (default: one name).
2. Server-reported state vocabulary, including whether an update racing provisioning yields Updating or an error (default: open on read, server adjudicates).
3. FEAT-p2 publication of the deployment schemas, the rollout representation, and the consumer-side error codes (including any terminal-state update refusal).

Note: per this run's write scope (feature directory only), no assumption records were created under `.sdlc/knowledge/`.

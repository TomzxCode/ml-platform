---
artifact: lifecycle
verdict: approved
reviewed_at: 2026-08-23
---

## Completeness

No blocking findings.
The one managed-and-tracked resource (Job) has a state diagram, per-state entry and exit conditions, transitions with trigger, actor, side effects, and guards, five invariants with enforcement and violation handling, and a retention row.
The author's pre-handoff self-check added two explicit points before this review: the job spec file is recorded as a user-authored input with no lifecycle entry, and the cancel-versus-completion race is documented as server-resolved.
Observation: the Job resource is a client view of a server-owned state machine; the ownership boundary is stated in the Overview and repeated in Out of Scope, which keeps this document from drifting into FEAT-p2 territory.

## Consistency

No issues found.
State names match the specification's open-on-read enum exactly (`queued`, `running`, `succeeded`, `failed`, `cancelled`).
The cancel transition corresponds to `POST /jobs/{job}/cancel` with the 409 `INVALID_STATE` guard from the specification's error mapping; the submit transition corresponds to the 201 in the submission sequences.
Event names (`job_submit_succeeded`, `job_submit_failed`, `job_cancelled`, `job_state_reported`) forward-reference telemetry.md, which is written in the same run with the same taxonomy.

## Spec Alignment

No issues found.
Every transition traces to a specification endpoint or sequence: submit to the happy-path and retry sequences, cancel to the cancel sequence, the read commands to the list, get, and logs sequences.
Dispatch and completion transitions are marked "Observed only", matching the constraint that the CLI never derives state.
Guard conditions restate the specification's constraints (local validation before submission, server-authoritative terminal-state refusal).

## Transition Correctness

No issues found.
Every state is reachable (Queued via submission; Running via dispatch; the three terminal states via their triggers) and the terminal states are dead ends by design.
The one transition the server may perform but the CLI does not model (queued to failed on a dispatch error) is called out in prose with the open-on-read tolerance, so it is a documented gap, not a missing one.
Backward transitions do not exist in the shared lifecycle, and the single race (cancel against completion) is documented with the server as the adjudicator.

## Invariant Soundness

No issues found.
Each invariant holds in every state: read-only rendering (including unknown states), cancel only via the server endpoint, stream closure at terminal state, one idempotency key per invocation, and names-only secret handling.
Enforcement mechanisms are specific (read-only handlers, output tests, retry tests, single cancel handler) and violation handling is realistic (test or review failure, matching the repository's gates).

## Retention Soundness

No issues found.
The client retains nothing between invocations, so the client-side retention row is a deliberate no-op with the expiry trigger (process exit) and cleanup action (nothing) stated.
Server-side job retention is explicitly out of scope with FEAT-p2 named as owner, which is the correct boundary for a client feature.

## Open Questions

Carried from the requirements and specification reviews; none are new to this artifact:

1. Log streaming transport, SSE versus gRPC (default: SSE in v1).
2. `job logs --follow` on a queued job (hold open versus exit with a note).
3. FEAT-p2 publication of the reference-confirmation and terminal-cancel error codes this client view assumes.

Note: per this run's write scope (feature directory only), no assumption records were created under `.sdlc/knowledge/`.

---
artifact: specification
verdict: approved
reviewed_at: 2026-08-23
---

## Ambiguities

No issues found.
Field names, types, constraints, validation order, error mapping, pagination, and streaming behavior are explicit.
The author's pre-handoff self-check tightened two points before this review: `compute.count` is optional with a default of 1 (the "Yes (default 1)" cell contradicted both the input spec.md and its own constraints column), and the `job submit` JSON document now carries `name` and `type` so the machine mode matches what the human line prints.

## Inconsistencies

No issues found.
All 8 mermaid blocks render successfully (validated with `npx -y @mermaid-js/mermaid-cli` and a no-sandbox puppeteer config; the sandbox failure without the config is an environment note, not a finding).
No `api.yaml` exists and none is required: the feature defines no API surface, and the consumer-side table cites the normative owner (FEAT-p2 contract) with an explicit drift rule (FEAT-p2 wins).
Data models match the contract table and the sequences: submit (201), reference failure (400), list (200 with cursor), get (200), logs (SSE), cancel (200), and the 401/404/409/5xx rows of the error mapping.
No orphan operations: every path in the summary table appears in a sequence or the error mapping, and vice versa.
The cli-design JSON shapes (`submit` and `list` carrying id, name, type, state; `cancel` carrying id and state) match the job summary model defined here.

## Incoherences

No issues found.
Technical decisions do not contradict each other; the closed `type` enum versus the open-on-read `state` enum is a deliberate asymmetry with recorded rationale (the discriminator selects local validation, while state is server-owned and must not crash older clients).
The SSE-in-v1 decision coheres with the reserved gRPC sketch and Open Question 4.

## Missing Information

No blocking findings.
Every FR and NFR from `requirements.md` is addressed: FR-2 via the deterministic validation order, FR-3 via the 400 reference-code rows, FR-4 via the filtered list and cursor behavior, FR-5/FR-6/FR-7 via the get, logs, and cancel sequences plus the 404 and 409 mappings, FR-8 via the shared-schema decision, FR-9 via secrets-as-names and the env-keys-only default, FR-10 via the idempotency-key sequence, NFR-1/NFR-2 via the inherited error model and retry mapping, NFR-3 via the names-only and keys-only rules, NFR-4 via the interactive-budget decision, NFR-5 via the testing-posture decision.
Authentication, workspace scoping, and the error model are inherited from FEAT-p2 and stated once.
Observations:

- `job get` has no dedicated sequence diagram of its own; its flow is covered inside the combined get-and-logs sequence (a single GET against a job path), acceptable for a read of one resource.
- Empty-result behavior for `job list` (exit 0, empty table or empty JSON array) is stated in cli-design.md; the specification's "complete array" wording implies the empty array but does not spell it out.
- The queued-job follow behavior (hold open versus exit with a note) remains intentionally unspecified pending Open Question 5; the stub will encode whichever default the plan phase picks.

## Implementability

No issues found.
All choices stay within cyclopts plus the shared client; the one long-lived SSE connection requires no new infrastructure.
The stub extension needed for Phase 2 testing (job endpoints plus an SSE log stream) is a plan-level deliverable, and risk 6 records the no-server posture with parent assumption 1.
External dependencies are explicit: the FEAT-p2 job endpoints and the two consumer-side error-code requests (risks 2 and 3).

## Reversibility

No issues found.
The feature holds no migrations, deployments, or persisted state; every change is code in the parent CLI package, revertible by commit.
The recorded defaults (env keys only, no cancel confirmation, SSE transport, `--json --follow` rejection) are each isolated behind a single handler or decision row, so reversing any of them is local.

## Forward Compatibility

No issues found.
Unknown spec-file fields are ignored with a warning; unknown response fields are tolerated; `state` is open on read so new server states render verbatim instead of failing; the scheme and type constants are extensible; `id` and `version` are treated as opaque.
The additive-only compatibility policy is inherited from the FEAT-p2 contract conventions.
Conscious tradeoff (not a gap): `type` is a closed v1 enum that rejects unknown values locally with an error listing accepted values, as mandated by the input spec; the drift risk between the CLI constant and the server's accepted types is tracked as risk 4.

## Open Questions

Carried from `requirements.md` and `review-requirements.md`, each with a recorded default in Technical Decisions where one was needed:

1. Does `code` also accept a source repository reference in addition to a container image?
2. Mount path semantics per input (passed through verbatim in v1).
3. `hyperparameters` value constraints (default: scalars and nested maps of scalars).
4. Log streaming transport, SSE versus gRPC (default: SSE in v1).
5. `job logs --follow` on a queued job (default chosen at plan time).
6. `env` echo in `job get` (default: keys only).

New from this specification:

7. FEAT-p2 must publish the reference-confirmation error codes and the terminal-cancel refusal code this consumer view assumes (risks 2 and 3).

Note: per this run's write scope (feature directory only), no assumption or decision records were created under `.sdlc/knowledge/`.
Questions 4, 6, and 7 carry the most implementation risk and are the first candidates for formal records when that path is writable.

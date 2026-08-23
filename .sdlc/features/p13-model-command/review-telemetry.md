---
artifact: telemetry
verdict: approved
reviewed_at: 2026-08-23
---

## Completeness

No issues found.
All three specification flows are instrumented end to end: register (success, local-validation, duplicate-version, auth, unreachable, server), list (success including empty, auth, unreachable, server), get (success, unknown-model, auth, unreachable, server), with failure reasons mapping onto the specification's error mapping and exit codes.
Every user-facing FR has events or properties (FR-1/FR-2 register events plus `version_explicit`, FR-3 list events with `result_count`, FR-4 get events with `version_count` and `had_lineage`, FR-5 `stage: local-validation` on failures, FR-6 `unknown-model` reason).
The funnel is a `flowchart TD` with one node per step; the three foreign steps (auth, consumption) are dashed with owners named, mirroring the FEAT-p3 and FEAT-p8 patterns, and the training-user path that bypasses register is explicitly noted as covered by the discovery metric instead of funnel ordering.
Counter metrics cover the four harm signals this group can produce (input ergonomics, bump-rule confusion, record-only strain, discoverability).

## Measurability

No issues found.
All five success metrics carry a concrete threshold, a computation method, and a timeframe.
Observations (non-blocking):

- "Discovery before first consumption" is computable only once FEAT-p10 and FEAT-p12 submission events exist; until then the metric is defined but pending, and the method column says so.
- "Time to first register" is conditional on installs that ever register, which is the correct denominator for an external-artifact workflow not every user needs.
- `had_lineage` is renderable directly from the response objects the get handler already holds, so the emission location is reliable.

## Actionability

No issues found.
Each counter metric names its investigation trigger and threshold; alert thresholds (30% local-validation, 20% duplicate-version, 20% unknown-model weekly; 50% local-path over two weeks) use weekly windows to avoid single-day noise, and the two-week local-path window deliberately avoids alerting on a one-off batch of external registrations.
The local-path alert's action is concrete: raise the upload-endpoint question with FEAT-p2 (specification OQ2 default).

## Consistency

No issues found.
Event names are snake_case and follow `model_<action>_<status>`, extending the FEAT-p3 `auth_<action>_<status>` and FEAT-p8 `compute_<action>_<status>` taxonomies without collision.
Reason values (`usage`, `duplicate-version`, `unknown-model`, `auth`, `unreachable`, `server`) match the specification's outcome vocabulary and error mapping exactly.
Property types and required flags are marked on every event.
No property carries a model name, artifact path or location, lineage job id, identity, or server URL, and the fingerprinting rationale is stated.

## Coverage Gaps

No issues found.
Error paths are instrumented, not just happy paths; cursor-following transparency has `pages_fetched`; the infrastructure requirements reuse FEAT-p3's install_id, opt-out gate, and local buffer rather than inventing a second pipeline.
The dashboard and three alerts are specified for the metrics that matter.
Observation (non-blocking): no `model register` progress events exist because registration is one request with no intermediate stages; if the future upload endpoint (OQ2 reversal) introduces transfer progress, its events belong to that scope, not this plan.

## Open Questions

1. The telemetry destination decision remains open platform-wide (inherited from FEAT-p3's telemetry plan); events buffer locally and nothing is sent until it is resolved.
2. "Discovery before first consumption" stays pending until FEAT-p10 and FEAT-p12 define their submission events; the join keys (install_id) already align.

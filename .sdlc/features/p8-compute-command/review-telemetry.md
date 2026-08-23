---
artifact: telemetry
verdict: approved
reviewed_at: 2026-08-23
---

## Completeness

No issues found.
Both specification flows are instrumented end to end: list (success including empty, auth, unreachable, server) and get (success, malformed-name, unknown-type, auth, unreachable, server), with failure reasons mapping onto the FR-5 exit codes.
Every user-facing FR has events or properties (FR-1/FR-2 command events, FR-3 failure reasons, FR-4 `json_mode`, FR-5 reason-to-exit mapping).
The funnel is a `flowchart TD` with one node per step; the two foreign steps (auth, job submission) are dashed with owners named, mirroring the FEAT-p3 pattern.
Counter metrics cover the three harm signals this group can produce (discoverability, quota exhaustion, reachability).

## Measurability

No issues found.
All three success metrics carry a concrete threshold, a computation method, and a timeframe.
Observations (non-blocking):
- "Discovery before first job" is computable only once FEAT-p9's job submission events exist; until then the metric is defined but pending, which the method column states honestly.
- A user may run `compute get` without ever running `compute list`; the funnel models the critical journey, not the only path, and the success metrics do not depend on strict funnel ordering.

## Actionability

No issues found.
Each counter metric names its investigation trigger and threshold; alert thresholds (20% unknown-type, 60% zero-remaining, 5% unreachable) are weekly windows, which avoids single-day noise.
The unreachable alert is shared with FEAT-p3's, which is correct since both point at the same server-side cause.

## Consistency

No issues found.
Event names are snake_case and follow `compute_<action>_<status>`, extending the FEAT-p3 `auth_<action>_<status>` taxonomy without collision.
Reason values (`malformed-name`, `unknown-type`, `auth`, `unreachable`, `server`) match the specification's outcome vocabulary; `zero_remaining` and `had_quota_limit` match the quota model's optional fields.
No property carries a type name, identity, server URL, or profile name, and the rationale (hardware fleet fingerprinting) is stated.

## Coverage Gaps

No issues found.
Error paths are instrumented, not just happy paths; the transparent pagination follow has `followed_pages`; the infrastructure requirements reuse FEAT-p3's install_id, opt-out, and local buffer rather than inventing a second pipeline.
The dashboard panel and two alerts are specified for the metrics that matter.
Observation (non-blocking): a `suggested` boolean on `compute_get_failed` (whether the server's closest match was present and rendered) would sharpen the unknown-type investigation; add it during implementation if the contract's error model settles on carrying the suggestion.

## Open Questions

1. The telemetry destination decision remains open platform-wide (inherited from FEAT-p3's telemetry plan); events buffer locally and nothing is sent until it is resolved.

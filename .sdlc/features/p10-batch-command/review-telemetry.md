---
artifact: telemetry
verdict: approved
reviewed_at: 2026-08-23
---

Resolved in this pass: the three rendering commands (`batch submit`, `batch list`, `batch get`) gained a `json_mode` property, closing the FR-12 coverage gap noted under Completeness.
Revision 1 (post-approval amendment): FEAT-p9's landed listing model (cursor-following to exhaustion) made `has_next_page` meaningless; it was replaced by `pages_fetched`. Re-reviewed: still approved.
Verification: the funnel flowchart renders (mmdc via npx).

## Completeness

No blocking findings after the amendment.
All five commands have events, including failure events with reasons; the funnel covers install to outcome with foreign steps dashed and owned (FEAT-p3).
Counter metrics cover the four harm signals (spec ergonomics, discoverability, feedback-loop gaps, runaway polling).
Observation (non-blocking): FR-9 (id equivalence) is a design property rather than usage, so it has no event; its health is watched indirectly through the fast-path-preference ratio.

## Measurability

No issues found.
Every success metric has a target, a computation method, and a timeframe; every event fires from a named command handler with primitive properties.
The state properties record server strings verbatim, so measurement survives future states.

## Actionability

No issues found.
Each counter metric names its concern and an investigation trigger; alert thresholds are weekly aggregates (not per-occurrence), avoiding noise.

## Consistency

No issues found.
Event names are snake_case in the `batch_<command>_<status>` pattern matching the command terminology in specification.md.
Properties are typed with required flags; the anonymous-by-construction rule is stated and extends FEAT-p3's baseline with no-resource-names.

## Coverage Gaps

No blocking findings.
Error paths are instrumented (`batch_submit_failed` with stage and reason, `batch_cancel_failed` with reason).
Server-side job execution metrics are explicitly out of scope (FEAT-p2); the deferred telemetry destination is inherited from FEAT-p3's plan rather than re-decided here, which is the right ownership.

## Open Questions

1. The fast-path-preference metric requires a `job_list_succeeded` event with its type filter recorded, owned by FEAT-p9's telemetry plan; without it the denominator is unmeasurable. Tracked as a cross-feature dependency, not blocking this plan.

---
artifact: telemetry
verdict: approved
reviewed_at: 2026-08-23
---

Revision 1 resolved the finding; the re-review below reflects the amended telemetry plan.

## Completeness

No issues found.
Every user-facing flow and requirement outcome is instrumented: login success and failure (with reason mapped to exit codes 1/2/3), logout, status (validity and check path), and first run.
The funnel is a rendered `flowchart TD` with one node per step; the dashed step 3 correctly marks ownership by the operation command groups.
Counter metrics cover harm signals (reachability, error-message quality, NFR-4 erosion).

## Measurability

No issues found.
Every success metric has a concrete target, computation method, and timeframe; events name their emission location; properties are typed with required flags.
Verification: the funnel flowchart renders (mmdc).

## Actionability

No issues found.
Each counter threshold names the concern it detects and the investigation it triggers; dashboard and alert conditions are stated even though creation is deferred with the destination.

## Consistency

No issues found.
Resolved in revision 1: the dangling "(Open Questions 1)" reference is replaced with a plain statement that the destination decision is pending.
Event names follow snake_case entity_action_status; event names and payload fields agree with lifecycle.md's Event Emissions (telemetry adds `prompt_used`, which lifecycle's payload column summarizes).

## Coverage Gaps

No issues found.
Error states are instrumented via `auth_login_failed.reason`; the deferred-destination infrastructure constraint is explicit, and opt-out is required before first emission.

## Open Questions

1. Where do client telemetry events go once leaving the machine (a platform-operated endpoint, or none in v1)? Instrumentation lands behind the local buffer either way, so implementation is not blocked.

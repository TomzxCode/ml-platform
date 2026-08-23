---
artifact: specification
verdict: approved
reviewed_at: 2026-08-23
---

Revision 1 added the parsed-spec logging decision (Missing Information finding below); the re-review reflects the amended specification.
Revision 2 (post-approval amendment): FEAT-p9's specification landed concurrently and chose transparent cursor-following to exhaustion (no paging flags) and a usage error for `--json --follow`; this feature adopted both, dissolving Inconsistencies item 2 and Open Questions 1 and 2 below, and resolving requirements Open Question 1. Re-reviewed against the revision: still approved.
Verification: all seven mermaid blocks render (mmdc via npx, puppeteer `--no-sandbox` config); no `api.yaml` exists to lint because the feature defines no API surface (consumer-only), matching the FEAT-p3 sibling pattern.

## Ambiguities

No issues found.
Field names and types are pinned per table; behavior descriptions name the actor, the action, and the exit code.
The `truncated` flag on the no-follow `batch logs --json` document is defined (bool) though its threshold (server-side truncation of very large stored logs) is owned by FEAT-p2; acceptable as a rendered pass-through.

## Inconsistencies

Findings (non-blocking, tracked):
- FR-12 ("exactly one valid JSON document") is unsatisfiable as written for `batch logs --follow`, which the specification resolves with a newline-delimited JSON exception, documented under Technical Decisions and tracked as Risk 4 for FEAT-p1 ratification. The specification is internally coherent; the fix belongs in the parent's output conventions when FEAT-p1's plan is next revised (outside this run's write scope).
- The pagination flag set (`--limit`, `--cursor`) is consistent within this document but depends on FEAT-p9 adopting the same flags (Risk 3); divergence would force re-scoping FR-5's equivalence clause.

## Incoherences

No issues found.
The facade decision, the idempotency-key derivation, and the exit-code mapping do not contradict each other; the Ctrl-C-exits-0 decision is justified against the reserved exit 4 rather than conflicting with it.

## Missing Information

Resolved in revision 1: the specification now states that the parsed spec object is never logged and that verbose diagnostics describe validation outcomes only, closing the NFR-3 gap (submitted specs carry `secrets` references and `env` values).
No further blocking findings.
Authentication (session resolution via FEAT-p3, 401 exits 2), error cases (unknown reference, unknown id, terminal cancel, unreachable server), and the NFR-4 local-path budget are all addressed; performance targets beyond NFR-4 are server-owned (FEAT-p2).

## Implementability

No issues found.
Every dependency is named with its owner and interface (validator and submission service: FEAT-p9; jobs client: FEAT-p1 client core; contract and transport: FEAT-p2; session: FEAT-p3).
Dependency direction is one-way (batch facade into shared machinery); no circularity.
The fresh-per-invocation idempotency key avoids the nightly-resubmission replay trap while keeping in-invocation network retries safe.

## Reversibility

No issues found.
The feature is client-side and additive: commands can be removed without migration, no persisted state is introduced, and every decision (pagination flags, NDJSON exception, exit-code mapping) is reversible without coordinated upgrades.

## Forward Compatibility

No issues found.
Consumers tolerate unknown fields in the job summary and in submitted specs; unknown `state` values render verbatim so future server-side states degrade to display; pagination and a future `--name` filter are additive extension points.
Minor observation: no CLI-local version field is introduced; compatibility piggybacks on FEAT-p2's additive-only contract policy, which is the right owner for it.

## Open Questions

1. Will FEAT-p9 adopt the `--limit`/`--cursor` flag set on `job list` verbatim? (Risk 3; blocks FR-5's final equivalence check, not implementation.)
2. Will FEAT-p1's output conventions ratify the NDJSON exception for streaming `--json`? (Risk 4; human-mode behavior is unaffected either way.)
3. Which transport does FEAT-p2 settle on for log streaming (gRPC versus SSE)? Confined to the jobs client when it changes.

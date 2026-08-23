---
artifact: specification
verdict: approved
reviewed_at: 2026-08-23
---

## Ambiguities

No blocking findings.
Field names, types, constraints, validation order, error mapping, pagination behavior, and the three JSON shapes are all explicit.
Observation: run `id` is described as the job identifier of the attached training run, which is ambiguous between the job's name and its server-assigned id; the client treats it as an opaque string it renders but never constructs, and FEAT-p2's run schema (risk 2) settles the semantics without a change here.

## Inconsistencies

No issues found.
All 6 mermaid blocks render successfully (validated with `npx -y @mermaid-js/mermaid-cli`; the default sandboxed launch fails on this host, `--no-sandbox` via a puppeteer config was required, a tooling note, not a finding).
No `api.yaml` exists and none is required: the feature defines no API surface, and the consumer-side table cites the normative owner (FEAT-p2 contract plan, which already pins the same three paths) with an explicit drift rule (FEAT-p2 wins).
Data models match the contract table and the sequences: create (201), list (200, cursor), get (200 with runs), and the 400/401/404/409/5xx rows of the error mapping.
No orphan operations: every path in the summary table appears in a sequence; the `POST /workspaces/{ws}/jobs` message in the run-attachment sequence is labeled "shown for context" and owned by FEAT-p9, so it is not an operation of this feature.

## Incoherences

No issues found.
Technical decisions do not contradict each other: the metrics-source decision (server run records only), the OQ2 default (fail on unknown name at submit), and the run-attachment sequence all state one coherent posture.
The architecture (command group composing the FEAT-p1 client core, read-only toward runs) matches the stated constraints (cyclopts, client-of-server, cross-cutting behavior owned by FEAT-p1).

## Missing Information

No blocking findings.
Every FR and NFR from `requirements.md` is addressed: FR-1 to FR-5 by the data models, contract table, sequences, and error mapping; NFR-1 via the validation-error and error-mapping rules; NFR-2 via the inherited retry policy; NFR-3 via the interactive-budget decision; NFR-4 via the testing-posture decision.
Authentication and authorization are inherited from the FEAT-p2 contract conventions and stated once.
Observation: `experiment get` and `experiment list` have no dedicated sequence for the empty case (no runs, no experiments); the empty cases are fixed by the pagination rule (a possibly-empty complete array) and the cli-design output rules, which is acceptable at this size.

## Implementability

No issues found.
All choices stay within cyclopts plus the shared client; no circular dependencies.
External dependencies are explicit: the FEAT-p2 experiment endpoints (not yet published; stub only, risks 1 and 5) and the FEAT-p9 attach-by-name semantics (risk 3).

## Reversibility

No issues found.
The feature holds no persisted state; every decision (OQ defaults, JSON shapes, validation order) is locally revisable.
No migrations, deployments, or destructive operations are introduced.

## Forward Compatibility

No issues found.
Unknown spec-file fields are ignored with a warning; unknown experiment and run response fields are tolerated and ignored; `metrics` is an opaque string-to-number map; the `description` key is omitted rather than `null` so a future type change stays additive.
No closed enums are introduced by this feature.
The additive-only compatibility policy is inherited from the FEAT-p2 contract conventions.

## Open Questions

Carried from `requirements.md`, each with a recorded default in Technical Decisions:

1. Metrics source: server run records only (no CLI log parsing) in v1.
2. Unknown experiment name at training submit: fail naming the experiment; decision owned with FEAT-p9.
3. Run pagination on `experiment get`: single response assumed in v1.

New from this specification:

4. Run `id` semantics (job name vs job id) and metric value types await the FEAT-p2 run and run-metrics schemas (risks 2 and 4).

Note: per this run's write scope (feature directory only), no assumption or decision records were created under `.sdlc/knowledge/`.
Questions 3 and 4 are the first candidates for formal records when that path is writable.

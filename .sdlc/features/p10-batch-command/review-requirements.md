---
artifact: requirements
verdict: approved
reviewed_at: 2026-08-23
---

Post-review amendment: FEAT-p9's landed pagination model (cursor-following to exhaustion, no paging flags) resolved Open Question 1; FR-5 was reworded accordingly and a Resolutions line was added recording the disposition of all six open questions (none blocks implementation). Verdict unchanged.

## Clarity

No issues found.
Each requirement names its subject (a specific `batch` subcommand) and action; vague terms are absent.
NFR-4's scope ("excluding server round trips") mirrors the parent's interactive-budget intent and is testable on the locally-answerable paths.
FR-3 intentionally says "exit non-zero" rather than fixing a code: the exit code for a server-confirmed unknown reference (usage-class 1 versus server-error 3) is a specifications-phase decision, recorded as Open Question 5 below.

## Completeness

No blocking findings.
Stakeholders represented (MLEs, ML infrastructure engineers, FEAT-p9 implementors); happy paths and error/edge cases covered (unparsable spec, missing field, wrong type, unknown model, unknown dataset, unknown state value, unknown job id, terminal-state cancel, idempotent resubmission); NFRs span usability, reliability, security, performance, portability, maintainability, plus NFR-7 pinning the no-fork constraint from the seeded spec.
Observations (non-blocking):
- `batch submit` with a nonexistent spec-file path is not called out explicitly; it is naturally covered by FR-1's local-validation duty and NFR-1's actionable-error duty, and the specifications phase should fix its message shape.
- `batch logs` on a job still queued (no lines produced yet) is unspecified; either empty output or a "no logs yet" notice satisfies FR-8, to be settled in specifications.
- The command surface is fixed by the seeded spec.md and the FEAT-p1 command reference, so no `cli-design.md` companion exists; the CLI Design category is therefore omitted from this pass.

## Testability

No issues found.
Every FR and NFR has at least one well-formed gherkin block whose tag matches its requirement ID.
NFR-4 carries a quantitative threshold (< 500 ms) scoped to local paths; NFR-5 is checkable by behavioral parity across platforms; NFR-7 is checkable by mutating the shared schema and running both suites.
FR-4's idempotent-resubmission scenario depends on an idempotency-key derivation the document does not fix; see Feasibility and Open Question 6.
Minor style note: a few `When` steps bundle two related actions (for example "runs batch list and job list --type batch-inference"); this matches the parent FEAT-p1 artifact style and stays parseable.

## Feasibility

Findings (non-blocking, tracked):
- The group depends on FEAT-p9's normative batch-inference spec schema and shared submission machinery, which is being specified concurrently; until FEAT-p9's specification lands, the code-level seam (module ownership of schema and validator) is provisional. Mitigated by NFR-7's no-fork requirement and by coding against the shared module from the start.
- The group depends on the FEAT-p2 API contract (submission with `Idempotency-Key`, filtered job listing, log streaming, cancellation); tests run against the contract-conformant stub per FEAT-p1 assumption 1 (`.sdlc/knowledge/assumptions/1-cli-target-server.md`).
- Whether `batch submit` derives a stable idempotency key (spec-content hash) or a fresh key per invocation decides whether FR-4's resubmission scenario replays or creates a second job; recorded as Open Question 6 rather than guessed.
- No new assumption records needed: none of the open questions blocks implementation, since submission, listing, inspection, streaming, and cancellation can ship against the stub while OQ 1 to OQ 6 resolve.

## Conflicts

| Requirements | Type | Description | Suggested Resolution |
|---|---|---|---|
| FR-5 vs Open Question 1 | Functional vs unresolved-design tension | FR-5 (Must) asserts output equality with `job list --type batch-inference`, while the pagination flag surface is not yet fixed; divergent paging flags between the two commands would break the equality. | Resolved in this pass: FR-5 was amended to scope equality to "the same paging parameters", so the two commands must share flag semantics once OQ 1 settles them; no requirement relaxation needed. |

## Open Questions

1. How is cursor pagination surfaced in `batch list` (flag names, default page size), given FEAT-p2's cursor-paginated list endpoints?
2. Does `batch get` on a succeeded job surface the registered output dataset, or only the job state?
3. Does `batch logs --follow` on an already-terminal job print stored logs then exit, or error?
4. Should `batch list` grow a `--name` filter for v1, or is state filtering enough?
5. Which exit code applies to a server-confirmed unknown reference (model or dataset): 1 (usage class) or 3 (server error)?
6. Does `batch submit` derive a stable idempotency key from spec content (replay on resubmit) or generate a fresh one per invocation (duplicate job on resubmit)?

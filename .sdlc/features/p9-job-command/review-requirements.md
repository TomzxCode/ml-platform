---
artifact: requirements
verdict: approved
reviewed_at: 2026-08-23
---

## Clarity

No issues found.
Each requirement names its subject and action; the validation rules in FR-2 enumerate the concrete checks (parse, per-type required fields, enums, structural reference forms), and the requirement that names the resource on every not-found path (FR-5, FR-6, FR-7) is stated consistently.
Minor note: FR-2's "structurally valid" is defined operationally by the acceptance scenarios (model `name:version`, kebab-case names) and fixed further by the cli-design validation order.

## Completeness

No blocking findings.
Stakeholders represented, including the FEAT-p10 consumer of the shared schema; happy-path and error cases covered for all five subcommands; NFRs span usability, reliability, security, performance, and maintainability.
Observations:

- All not-found paths now name the resource: the author's pre-handoff self-check tightened FR-6 (unknown id on `job logs`) to match FR-5 and FR-7, and split FR-3's packed scenario into single-purpose scenarios (model, secret, compute type).
- Empty-result behavior for `job list` (exit 0, empty table or array) is defined in cli-design.md only; the specification phase should carry it into the output contract, as it did for p8.
- `env` value handling in `job get` output is deliberately left to Open Question 6 with a safe default (keys only) recorded in cli-design.md.

## Testability

No issues found.
Every FR and NFR has at least one well-formed gherkin block with a tag matching its requirement ID.
NFR-4 carries a quantitative threshold (under 500 ms) and an explicit exclusion (streaming follow mode).
FR-2's and FR-3's "no server call is made" steps are verifiable against the contract-conformant stub.
FR-8's scenario is code-level verifiable (shared module import, no forked copy), which matches its maintainability intent.

## Feasibility

Findings (non-blocking, tracked):

- All requirements code against the FEAT-p2 server contract; until that server exists they are testable only against a contract-conformant stub, the same posture as the parent feature (assumption 1, `.sdlc/knowledge/assumptions/1-cli-target-server.md`).
- FR-6's follow mode depends on the log streaming transport the FEAT-p2 contract defines (SSE endpoint plus gRPC sketch, contract plan Phase 3); the consumer-side choice is tracked as Open Question 4.
- FR-10 (retry-safe submission) depends on the FEAT-p2 idempotency convention (`Idempotency-Key` replay), a Phase 1 deliverable of the contract plan; the dependency is real but already scheduled on the owner's side.

## CLI Design

No blocking findings.
Every user-facing FR and NFR maps to at least one command or option in the traceability table; every command has a synopsis, typed options with defaults, at least one example, and an error table; exit codes, the stdout/stderr split, and machine-readable output are defined once and applied consistently.
Naming follows the FEAT-p1 conventions (kebab-case, verb-first subcommands under the `job` noun).
Example sessions demonstrate a happy path, a local validation error, a server-side reference error, and machine-readable consumption.
Observations:

- `job cancel` carries no confirmation prompt; the recorded rationale (a cancelled job is recoverable by resubmitting the same spec) and the escalation path (cli-design Open Question 5) make this a conscious default rather than an oversight.
- The `--json` plus `--follow` rejection is a deliberate exception to the parent's JSON-everywhere rule, recorded under Conflicts below and as cli-design Open Question 2.
- The interrupt-exits-0 decision for follow streams sets a convention that FEAT-p10's `batch logs` inherits; it is flagged for parent-surface alignment (cli-design Open Question 3).

## Conflicts

| Requirements | Type | Description | Suggested Resolution |
|---|---|---|---|
| FEAT-p1-FR-12 vs FR-6 | Potential contradiction | The parent requires machine-readable JSON output for every command when requested, but an unbounded follow stream cannot emit exactly one valid JSON document; the design rejects `--json --follow`. | Already captured as cli-design Open Question 2 and requirements Open Question 4's transport question; if the parent surface owner insists on JSON everywhere, switch follow mode to newline-delimited JSON events via a parent-surface change. |

## Open Questions

Carried from `requirements.md`:

1. Does `code` also accept a source repository reference in addition to a container image?
2. What are the mount path semantics per input, and does the FEAT-p2 contract expose them?
3. Is `hyperparameters` constrained to string and number values, or is any YAML value accepted?
4. Which log streaming transport does the CLI consume in v1: REST SSE, gRPC, or both?
5. Does `job logs --follow` on a queued job hold the stream open until lines appear, or exit with a note?
6. Does `job get` echo the submitted spec's `env` values, and if so how are sensitive values protected?

Added by the CLI design (interface-level):

7. Should `job submit` offer `--dry-run` (validate only)?
8. Is exit 0 on interrupting a follow stream the right cross-group convention?
9. What is the job id shape, and can the CLI validate its form locally?
10. Should `job cancel` carry a `--yes` confirmation flag if the parent mandates prompts on mutating commands?

Note: per this run's write scope (feature directory only), no assumption records were created under `.sdlc/knowledge/`.
Open Questions 4 and 6 carry the most implementation risk and are the first candidates for `/create-assumption` when that path is writable.

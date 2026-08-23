---
artifact: requirements
verdict: approved
reviewed_at: 2026-08-23
---

## Clarity

No issues found.
Each requirement names its subject and action; FR-2 enumerates the concrete scaling checks (mutual exclusion, bound pairing, min less than or equal to max, positive integers), and the not-found paths name the resource consistently (FR-5, FR-7).
The author's pre-handoff self-check tightened FR-2 to state the bound-pairing rule explicitly (the cli-design error table already enforced it) and split FR-8's two-action When into a single action with an observation Then.
Minor note: FR-3's "version per the registry convention" is defined operationally by the acceptance scenario and the cli-design form check, and pinned to the numeric default by Open Question 6.

## Completeness

No blocking findings.
Stakeholders represented, including the FEAT-p2 contract owner as an explicit consumer; happy-path and error cases covered for all five subcommands; NFRs span usability, reliability, performance, and maintainability, with security inherited wholesale from FEAT-p1 (this group carries no secret values, so no local security NFR is warranted).
Observations:

- All five parent FR-10 scenarios from FEAT-p1 (manage deployments, static count clears autoscaling, bounds update with min greater than max rejected) have local counterparts, and the group adds create-side and retry-safety scenarios the parent leaves implicit.
- The default scaling when every flag is omitted (static count of 1) and the empty-update rejection are recorded in cli-design.md only; the specification phase should carry both into the data model and output contract, as p8 and p9 did for their defaults.
- The one-name design (the endpoint name also addresses the deployment) is an interface decision recorded in cli-design.md with confirmation risk tracked as Open Question 4.

## Testability

No issues found.
Every FR and NFR has at least one well-formed gherkin block with a tag matching its requirement ID, single-action Whens, and "no server call is made" steps verifiable against the contract-conformant stub.
NFR-3 carries a quantitative threshold (under 500 ms) and NFR-4 names the four gates.
FR-2's four scenarios each isolate one rule, so the first-failure-deterministic validation order is directly testable.

## Feasibility

Findings (non-blocking, tracked):

- All requirements code against the FEAT-p2 deployment endpoints; until that server exists they are testable only against a contract-conformant stub, the same posture as the parent feature (assumption 1, `.sdlc/knowledge/assumptions/1-cli-target-server.md`).
- FR-8 depends on the server exposing rollout progress in the deployment view (a rollout object or equivalent); the `rollout` shape in cli-design.md is a consumer-side request until the contract lands.
- FR-9 (retry-safe create) depends on the FEAT-p2 idempotency convention (`Idempotency-Key` replay), a Phase 1 deliverable of the contract plan; the dependency is already scheduled on the owner's side.
- The rollout and deployment state vocabulary (`creating`, `serving`, `updating`, `stopped` as used in the example sessions) is illustrative until FEAT-p2 publishes the enum; the open-on-read rendering rule keeps unknown values safe.

## CLI Design

No blocking findings.
Every user-facing FR and NFR maps to at least one command or option in the traceability table; every command has a synopsis, typed options with defaults, at least one example, and an error table; exit codes, the stdout/stderr split, and machine-readable output are defined once and applied consistently.
Naming follows the FEAT-p1 conventions (kebab-case, verb-first subcommands under the `deploy` noun), and the command set matches the parent plan exactly (no invented flags or filters).
Example sessions demonstrate a happy path spanning create, list, get, and update with a rollout, a local validation error, and machine-readable consumption.
Observations:

- `deploy stop` carries no confirmation prompt; the recorded rationale (redeploying is one create away) and the escalation path (Open Question 2) make this a conscious default in the p9 no-prompt precedent, not an oversight.
- The one-name design (endpoint name addresses the deployment) is a real interface commitment recorded as a design principle with its confirmation risk tracked as Open Question 4; if FEAT-p2 assigns distinct names, create's output and every name-taking subcommand change in one place each.
- The `scaling` object shape (`static` versus `autoscale` modes) is a machine-output design intent pending the FEAT-p2 contract, consistent with how p9 recorded its job object shape.

## Conflicts

| Requirements | Type | Description | Suggested Resolution |
|---|---|---|---|
| FR-8 vs Constraints (server authority) | Potential contradiction | FR-8 asserts the endpoint keeps serving during a rollout, a server-side guarantee the CLI can only observe, while the constraints fix the CLI as a reporter that never derives state. | Read FR-8's continuity clause as an observed outcome: the acceptance scenario verifies it through `deploy get` observations only; carry the serving-continuity expectation to the FEAT-p2 contract as a consumer-side request (already implied by `architecture.md`'s online inference flow). No wording change required; the qualification is recorded here. |

## Open Questions

Carried from `requirements.md`:

1. Should rollout progress stream during `deploy update`, or remain visible only via `deploy get`?
2. Should `deploy stop` require confirmation or a `--yes` flag?
3. Does `deploy create` return as soon as the server accepts the request, or wait for endpoint readiness?
4. What is the relationship between the deployment name accepted by `deploy get`, `deploy update`, and `deploy stop` and the endpoint reference printed by `deploy create`?
5. Which state values does the server report for deployments and rollouts in v1, and should the CLI render unknown state values verbatim (default) or reject them?
6. Is the version part of a model reference numeric (as the p9 spec schema assumes), or are arbitrary version strings allowed?

Added by the CLI design (interface-level):

7. Is the one-name design right, or does the server assign a distinct deployment name the CLI must print and accept (mirrors question 4; the design principle records the v1 default)?
8. Should the default scaling when every flag is omitted be a static count of 1, or should create require an explicit scaling choice?

Note: per this run's write scope (feature directory only), no assumption records were created under `.sdlc/knowledge/`.
Questions 4 and 5 carry the most implementation risk and are the first candidates for `/create-assumption` when that path is writable.

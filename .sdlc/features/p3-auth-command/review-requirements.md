---
artifact: requirements
verdict: approved
reviewed_at: 2026-08-23
---

## Clarity

No issues found.
Each requirement names its subject (a specific `auth` subcommand or the storage mechanism) and action; vague terms are absent.
NFR-4's conditional clause ("when the session check is served without a server round trip") is precise and intentionally scopes the threshold; see Conflicts for the tension it resolves.

## Completeness

No blocking findings.
Stakeholders represented (MLEs, ML infrastructure engineers, FEAT-p1 Phase 2 implementors); error and edge cases covered (empty token, malformed URL, rejected credentials, unreachable server, idempotent logout, expired session, non-interactive without `--token`); NFRs span security, reliability, usability, performance, portability, maintainability.
Observations (non-blocking):
- Re-login while already logged in is not called out; the natural reading of FR-1 ("store only on success") is silent overwrite, and the specifications phase should confirm that reading.
- The masked prompt's exact behavior (hidden input versus per-character masking) is left to specifications; either satisfies FR-2.

## Testability

No issues found.
Every FR and NFR has at least one well-formed gherkin block with a tag matching its requirement ID.
NFR-4 carries a quantitative threshold (< 500 ms) scoped to the locally-answerable path.
NFR-5 is checkable by inspecting file permissions per platform; NFR-1 by inspecting output, logs, and JSON documents.

## Feasibility

Findings (non-blocking, tracked):
- The group depends on the FEAT-p2 API contract (a token-validation endpoint returning identity and, ideally, expiry); until that server exists, tests run against the contract-conformant stub, per FEAT-p1 assumption 1 (`.sdlc/knowledge/assumptions/1-cli-target-server.md`).
- What identity fields the validation endpoint returns (cli-design Open Question 3) gates the exact `auth status` output; any minimal identity field set keeps FR-5 satisfiable.
- No new assumption records needed: none of the open questions blocks implementation, since login, logout, and status can ship with the local-expiry default while OQ 2 is resolved.

## CLI Design

No issues found.
All eight FRs and six NFRs map in the traceability table; every command has a synopsis, typed options with defaults, examples, and an error table.
Exit codes (0/1/2/3/4), the stdout/stderr split, and `--json` shapes are defined and consistent with the FEAT-p1 plan's scheme.
Naming follows the FEAT-p1 conventions (`mlx`, kebab-case, noun group plus verbs); `logout` without confirmation is justified in the artifact (re-login is one command and the server URL is kept).
Example sessions cover happy paths (interactive, CI plus piping) and error paths (rejected credentials, unreachable server).
Minor note: the unreachable-server transcript's single `retrying (2/3)` line matches FEAT-p1 NFR-2's "at most one informational line"; keep to one line when implementing.

## Conflicts

| Requirements | Type | Description | Suggested Resolution |
|---|---|---|---|
| FR-5 vs NFR-4 | Functional vs non-functional tension | Session-validity checking may need a server round trip, which cannot be bounded by the 500 ms interactive budget; the parent FEAT-p1-NFR-4 lists `status` unconditionally. | Already reconciled in the document: NFR-4 is scoped to the locally-answerable path and FR-8 provides it (stored expiry), with the remainder tracked as Open Question 2. |

## Open Questions

1. Token expiry and refresh behavior on the first command after expiry (re-prompt versus hard failure); assumed the server returns an expiry the CLI honors (FEAT-p1 Phase 2 deliverable).
2. Does `auth status` validate with a server round trip or trust the locally stored expiry? Bounds NFR-4 and offline behavior.
3. Should `auth login` also read the token from `MLX_TOKEN` (cli-design lists it as proposed, not v1)?
4. Should `auth status` exit 2 on a missing or invalid session (script-friendly, as designed) or exit 0 and report state only?
5. What identity fields does the FEAT-p2 validation endpoint return, and which does `auth status` print?

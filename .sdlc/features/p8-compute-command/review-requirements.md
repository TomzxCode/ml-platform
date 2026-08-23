---
artifact: requirements
verdict: approved
reviewed_at: 2026-08-23
---

## Clarity

No issues found.
Each requirement names its subject (a specific `compute` subcommand, the local validation rule, or the output mode) and action; vague terms are absent.
NFR-3's "excluding server response time" clause deliberately scopes the threshold; see Conflicts for the tension it resolves.

## Completeness

No blocking findings.
Stakeholders represented (MLEs authoring job specs, ML infrastructure engineers preempting quota support requests, FEAT-p1 Phase 3 implementors); error and edge cases covered (malformed and unknown type names, authentication failure, unreachable server, JSON modes for both commands); NFRs span usability, reliability, performance, maintainability.
Observations (non-blocking):
- Empty-result behavior (exit 0, empty table or empty JSON array) is defined in cli-design.md only; the specification phase should carry it into the output contract.
- No security NFR is stated, correctly: the group holds no secrets, and bearer-token hygiene is inherited from the shared API client (FEAT-p1 Phase 2 deliverable).
- Whether zero-quota types appear in listings is left to Open Question 3; cli-design's example (gpu-h100-80gb at 0 of 2) implies they do.

## Testability

No issues found.
Every FR and NFR has at least one well-formed gherkin block with a tag matching its requirement ID.
NFR-3 carries a quantitative threshold (< 500 ms) scoped to the locally-controllable path.
FR-3's two scenarios cleanly split local format rejection from server-side unknown-name resolution, each independently checkable against the stub.

## Feasibility

Findings (non-blocking, tracked):
- The group depends on the FEAT-p2 API contract (compute type list and get endpoints, quota fields, closest-match error payload); until that server exists, tests run against the contract-conformant stub, per FEAT-p1 assumption 1 (`.sdlc/knowledge/assumptions/1-cli-target-server.md`).
- The closest-match suggestion is assumed to arrive in the shared error model's suggested-fix field (FEAT-p2 API conventions); if the contract drops it, the fallback is a client-side closest match over the list results, at the cost of an extra round trip.
- No new assumption records needed: none of the three open questions blocks implementation, since both commands can ship with the remaining-only quota display and no filters while they resolve.

## CLI Design

No issues found.
All five FRs and four NFRs map in the traceability table; both commands have a synopsis, typed options with defaults, examples, and an error table.
Exit codes (0/1/2/3/4), the stdout/stderr split, and `--json` shapes are defined and consistent with the FEAT-p1 plan's scheme; naming (`compute`, `list`, `get`) matches the parent plan's command tree verbatim.
The group is read-only, so no confirmation prompts are needed; the design says so explicitly.
Example sessions cover happy paths (table, key-value, JSON piping) and error paths (unknown type with closest match, malformed name), with exit codes shown.
Minor notes:
- Mapping an unknown-but-well-formed type name to exit 1 (usage class) even though detection is server-side is a deliberate choice; the specification phase should preserve it in the exit mapping table.
- The human table's "X of Y" quota cell reads as remaining-of-limit; cli-design Open Question 4 tracks whether to also show used. Keep the reading consistent when implementing.

## Conflicts

| Requirements | Type | Description | Suggested Resolution |
|---|---|---|---|
| NFR-3 vs parent FEAT-p1-NFR-4 | Functional vs non-functional tension | The parent lists `list` commands unconditionally under 500 ms, but compute results cannot be cached locally because the server is authoritative, so the budget cannot cover the round trip. | Already reconciled in the document: NFR-3 scopes the budget to local startup and rendering, excluding server response time. |
| FR-2 vs parent plan note ("quota limits live in `quota get`") | Cross-artifact scoping note | The FEAT-p1 plan's compute get row says quota limits live in the `quota` group, while FEAT-p1-FR-18 and this feature's spec require per-type quota display. | Already reconciled in Constraints: limit administration belongs to `quota` (FEAT-p1-FR-21); this group reports availability and remaining quota per type. No requirement text conflicts. |

## Open Questions

1. Should `compute list` support filters (for example GPU-only) in v1, or defer them?
2. If the FEAT-p2 list endpoint paginates with cursors, does `compute list` follow cursors transparently at the expected cardinality?
3. What quota fields does the contract return per type (remaining only, or used, limit, and remaining), and how are types without quota presented?
4. Should the human table's QUOTA column show remaining only or used-of-total (cli-design Open Question 4)?

None of the open questions carries implementation-blocking risk, so no `/create-assumption` records are required; writing to `.sdlc/knowledge/` is also outside this run's feature-directory-only write scope.

# ADR-060 — Intake must-see chips from tool loop only (no forced nominate)

- **Status:** Accepted
- **Date:** 2026-09-09
- **Deciders:** product + agent

## Context

`agent-itinerary-95` forced `nominateMustSeeViaLlm` before the intake tool loop so Q3 always had chips. That guarantees chips but is not a pure true-agent loop: code searches for the model.

Product clarified (merged into [`agent-design.md`](../agent-specs/agent-design.md) MVP-T1 as-built): takeoff *can* produce chips via the loop; the open choice is whether to *guarantee* them. Decision: **do not guarantee** — prefer purity; Q3 may be empty with hand-type + refetch.

## Decision

1. Remove `runForcedNominate` from `plan_trip` intake.
2. Chips come only from the intake tool loop (`geocode` → `search_places` → `commit_trip` → `ask_user`), plus existing trip `must_see` reload on same `trip_id` (locale switch).
3. System prompt *encourages* search before `ask_user`; does not enforce it in code.
4. Keep `nominateMustSeeViaLlm` on `plan_itinerary` (full schedule path), not takeoff intake.
5. 2play Q3: empty i18n state, free-text entry, optional `/api/plan/candidates` refetch — no automatic third `plan_trip`.

## Consequences

- Q3 may show zero chips if the model asks first or search returns empty.
- Intake latency and LLM/vendor cost drop when nominate was the dominant pre-loop call.
- AC1 of `agent-itinerary-95` superseded by `agent-itinerary-96`; cluster dedupe / limit 8 / ADR-042 remain.

## Alternatives considered

| Option | Why not |
| --- | --- |
| Keep forced nominate | Guarantees Q3 chips but violates pure-agent preference |
| Soft force (timeout then nominate) | Still a host-side search; complexity without purity |

## Related

- Stories: `agent-itinerary-96`, superseded AC1 of `agent-itinerary-95`
- ADR-042, ADR-054, ADR-059

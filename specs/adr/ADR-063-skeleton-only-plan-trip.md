# ADR-063 — `skeleton_only` on plan_trip (where2play T3)

- **Status:** Accepted
- **Date:** 2026-09-10
- **Related:** [ADR-062](./ADR-062-mvp-t3-skeleton-vs-t4-nominate.md) · `agent-itinerary-100` · `2play-plan-101`

## Context

MVP-T3 needs `plan_trip` to create a trip, make/commit skeleton, emit phases, and **stop** — without fixed 4Q `need_input` and without `plan_next_stop` fill.

MCP / ChatBox / legacy callers still expect the prior intake + full-loop behavior when bounds are incomplete or when a full filled itinerary is desired.

## Decision

where2play T3 BFF sends **`skeleton_only: true`** on `plan_trip`.

| Caller | Flag | Behavior |
| --- | --- | --- |
| where2play after Takeoff 11 submit | `skeleton_only: true` | Persist takeoff bounds (incl. `party_size` / `start_time` / `other`); phases `trip_created` → `skeleton_generating` → `skeleton_ready`; stop after skeleton; `ready` **without** requiring `filledStops.length > 0`; no fixed 4Q |
| MCP / other | omit flag | Unchanged intake / full-loop semantics |

HTTP schema and dispatch must forward `party_size`, `start_time`, `other`, and `skeleton_only` (do not strip).

## Alternatives considered

1. **Always skeleton-stop for all callers** — rejected; breaks MCP full-loop demos and T1 paths.
2. **Separate tool `plan_trip_skeleton`** — rejected; duplicates surface; BFF already owns the product path.
3. **`stop_after: "skeleton"` string enum** — equivalent; boolean `skeleton_only` is enough for T3.

## Consequences

- 2play must not interpret T3 responses as fixed four-question intake.
- Success criteria for T3 readiness is a fetchable skeleton, not filled stops.
- Full fill remains a later slice (T5+) or non-`skeleton_only` callers.

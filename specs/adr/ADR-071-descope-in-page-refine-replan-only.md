# ADR-071: Descope in-page chat refine — replan only

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Accepted**（2026-09-18）

## Context

MVP-T9 shipped `plan_trip` refine + where2play `/api/chat` (ADR-050, ADR-070). Live probes (Shanghai family) showed:

- **Probe 1** (vague「改近一点」): clarify works — good UX for true-agent, but product chose simpler scope.
- **Probe 2** (slot-complete morning replace): **unstable** — search empty, false success copy, full `days[]` rewrite noise (`refill_day_indices` bloated).

Product decision: **remove chat-based itinerary edits**; travelers who want a different trip use **Replan** (confirm → new full-loop). Planning **need_input** composer (hotel, expand_radius, etc.) stays.

## Decision

### D1 — Cancel T9 in-page refine (full-stack)

Remove:

- where2play post-complete composer + refine thread + `POST /api/chat` refine
- places-agent `plan_trip` refine mode (`plan-trip-refine.ts`, `refine` body field)

Keep:

- Takeoff → T3 assistant progress → fill → complete
- **Replan** (`ReplanDialog`, soft replan chip, `play.plan.replan*`)
- **need_input** composer during planning (not after complete)

### D2 — Supersede ADR-070

[ADR-070](./ADR-070-refine-skeleton-true-agent-loop.md) **Cancelled** — skeleton refine loop not pursued.

### D3 — what2eat unchanged

2eat list/place chat and `/api/chat` on what2eat are out of scope.

### D4 — Saved chat snapshot (T10)

`2play-plan-25` AC2–3 / saved DB chat deferred; no refine transcript to persist after descope.

## Consequences

**Positive**

- Simpler product: one path to change trip (replan).
- Less LLM cost and failure surface on post-complete chat.
- UI matches capability: no composer promising edits that are removed.

**Negative**

- No incremental tweak without full replan (user cost).
- Removed agent refine code; reintroducing later needs new story + ADR.

## Implementation（2026-09-18）

Full-stack descope landed in `where2play` + `places-agent`:

- Removed post-complete composer, refine thread, `POST /api/chat`, agent `plan_trip` refine.
- Kept Replan dialog + need_input composer during planning.
- Vitest regression green for thread ordering, i18n catalog, T3 hydrate, no `plan-nav-input` after complete.

## Verification

| Gate | Status |
| --- | --- |
| Unit / component (vitest) | **Pass** — descope-related suites |
| Manual browser (takeoff → plan → complete → Replan) | **Unblocked** — map quota restored 2026-09-20; **not yet run** |
| DoD usable confirm | **Pending** — quota no longer blocking |

Until live verify: treat descope as **implemented, not usability-confirmed**.

## References

- Probe: [`shanghai-family-refine-probe.md`](../knowledge/agent/shanghai-family-refine-probe.md)
- Close notes: [`where2play-t9-close-lessons.md`](../knowledge/web-app-development/where2play-t9-close-lessons.md)
- Supersedes: ADR-070, MVP-T9 refine stories
